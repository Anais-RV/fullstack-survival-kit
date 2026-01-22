# Conceptos Fundamentales de Bases de Datos

> **Tablas, filas, columnas, claves y relaciones**

---

## Introducción

En este módulo aprenderás el **vocabulario** y **conceptos** que necesitas para entender bases de datos relacionales.

---

## 1. Tabla (Table)

**Definición:**  
Una **tabla** es una estructura que almacena datos en filas y columnas.

**Analogía:**  
Como una hoja de Excel.

```
Tabla: usuarios
+----+-----------+-------------------+------+
| id | nombre    | email             | edad |
+----+-----------+-------------------+------+
|  1 | Ana García| ana@example.com   |   25 |
|  2 | Bob Smith | bob@example.com   |   30 |
|  3 | Carlos    | carlos@example.com|   28 |
+----+-----------+-------------------+------+
```

**Características:**
- Representa una **entidad** (usuarios, productos, posts)
- Tiene un **nombre** único
- Define **columnas** (estructura)
- Contiene **filas** (datos)

---

## 2. Fila / Registro / Tupla (Row / Record / Tuple)

**Definición:**  
Una **fila** es un registro individual de datos.

```
Tabla: usuarios
+----+-----------+-------------------+------+
| id | nombre    | email             | edad |
+----+-----------+-------------------+------+
|  1 | Ana García| ana@example.com   |   25 |  ← FILA
+----+-----------+-------------------+------+
```

**Términos sinónimos:**
- **Fila** (row) ← más común
- **Registro** (record)
- **Tupla** (tuple) ← término matemático

**Ejemplo:**  
"La base de datos tiene 10,000 filas" = 10,000 usuarios

---

## 3. Columna / Campo / Atributo (Column / Field / Attribute)

**Definición:**  
Una **columna** es un atributo de la entidad.

```
Tabla: usuarios
+----+-----------+-------------------+------+
| id | nombre    | email             | edad |  ← COLUMNAS
+----+-----------+-------------------+------+
|  1 | Ana García| ana@example.com   |   25 |
+----+-----------+-------------------+------+
     ↑
   COLUMNA
```

**Cada columna tiene:**
- **Nombre** (ej: `email`)
- **Tipo de dato** (ej: `VARCHAR`, `INTEGER`)
- **Restricciones** (ej: `NOT NULL`, `UNIQUE`)

---

## 4. Esquema (Schema)

**Definición:**  
El **esquema** es la estructura de la tabla (qué columnas tiene y qué tipos).

```sql
-- Esquema de la tabla usuarios
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    edad INTEGER,
    fecha_registro DATE DEFAULT CURRENT_DATE
);
```

**Componentes del esquema:**
- Nombre de cada columna
- Tipo de dato
- Restricciones (claves, unique, not null)

---

## 5. Tipo de Datos (Data Types)

Cada columna debe tener un **tipo de dato**.

### Tipos numéricos

```sql
INTEGER        → Números enteros (1, 2, 100)
BIGINT         → Enteros grandes
DECIMAL(10,2)  → Decimales exactos (precio: 19.99)
FLOAT          → Decimales aproximados
```

### Tipos de texto

```sql
CHAR(5)        → Texto fijo (siempre 5 caracteres)
VARCHAR(100)   → Texto variable (hasta 100 caracteres)
TEXT           → Texto largo sin límite
```

### Tipos de fecha/hora

```sql
DATE           → Fecha (2024-01-01)
TIME           → Hora (14:30:00)
TIMESTAMP      → Fecha y hora (2024-01-01 14:30:00)
```

### Tipos booleanos

```sql
BOOLEAN        → true/false
```

**Ejemplo completo:**

```sql
CREATE TABLE productos (
    id INTEGER,               -- Número
    nombre VARCHAR(200),      -- Texto variable
    descripcion TEXT,         -- Texto largo
    precio DECIMAL(10,2),     -- Decimal (precio)
    stock INTEGER,            -- Entero
    activo BOOLEAN,           -- true/false
    fecha_creacion TIMESTAMP  -- Fecha y hora
);
```

---

## 6. Clave Primaria (Primary Key)

**Definición:**  
Un **identificador único** para cada fila.

```
Tabla: usuarios
+----+-----------+-------------------+
| id | nombre    | email             |  ← id es PRIMARY KEY
+----+-----------+-------------------+
|  1 | Ana       | ana@example.com   |
|  2 | Bob       | bob@example.com   |
+----+-----------+-------------------+
 ↑
PRIMARY KEY (único e irrepetible)
```

**Características:**
- ✅ **Único** (no se repite)
- ✅ **No nulo** (siempre tiene valor)
- ✅ **Inmutable** (no cambia)

**Ejemplo:**

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,  -- ← Clave primaria
    nombre VARCHAR(100)
);

INSERT INTO usuarios (id, nombre) VALUES (1, 'Ana');
INSERT INTO usuarios (id, nombre) VALUES (1, 'Bob');  -- ❌ ERROR: id duplicado
```

---

## 7. Clave Foránea (Foreign Key)

**Definición:**  
Una **referencia** a la clave primaria de otra tabla.

```
Tabla: usuarios                    Tabla: posts
+----+-----------+                 +----+------------+-----------+
| id | nombre    |                 | id | usuario_id | titulo    |
+----+-----------+                 +----+------------+-----------+
|  1 | Ana       |  ←───────────── |  1 |          1 | Post 1    |
|  2 | Bob       |  ←───────────── |  2 |          1 | Post 2    |
+----+-----------+                 |  3 |          2 | Post 3    |
                                   +----+------------+-----------+
                                          ↑
                                    FOREIGN KEY
```

**Características:**
- ✅ Conecta dos tablas
- ✅ Debe existir en la tabla referenciada
- ✅ Garantiza **integridad referencial**

**Ejemplo:**

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100)
);

CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    usuario_id INTEGER,
    titulo VARCHAR(200),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);

-- ✅ Válido (usuario 1 existe)
INSERT INTO posts (id, usuario_id, titulo) VALUES (1, 1, 'Post');

-- ❌ ERROR (usuario 999 no existe)
INSERT INTO posts (id, usuario_id, titulo) VALUES (2, 999, 'Post');
```

---

## 8. Tipos de Relaciones

### 8.1 Relación Uno a Muchos (1:N)

**Descripción:**  
Un registro de una tabla se relaciona con **muchos** de otra.

```
usuarios (1) ──── (N) posts

Un usuario tiene muchos posts
```

**Ejemplo:**

```
Tabla: usuarios
+----+-----------+
| id | nombre    |
+----+-----------+
|  1 | Ana       |
+----+-----------+

Tabla: posts
+----+------------+-----------+
| id | usuario_id | titulo    |
+----+------------+-----------+
|  1 |          1 | Post 1    |  ← Ana
|  2 |          1 | Post 2    |  ← Ana
|  3 |          1 | Post 3    |  ← Ana
+----+------------+-----------+
```

**Casos comunes:**
- Usuario → Posts
- Categoría → Productos
- Cliente → Pedidos

---

### 8.2 Relación Muchos a Muchos (N:M)

**Descripción:**  
Múltiples registros de una tabla se relacionan con múltiples de otra.

```
estudiantes (N) ──── (M) cursos

Un estudiante toma muchos cursos
Un curso tiene muchos estudiantes
```

**Implementación: Tabla intermedia**

```
Tabla: estudiantes            Tabla: cursos
+----+-----------+             +----+-----------+
| id | nombre    |             | id | nombre    |
+----+-----------+             +----+-----------+
|  1 | Ana       |             |  1 | Python    |
|  2 | Bob       |             |  2 | JavaScript|
+----+-----------+             +----+-----------+
        │                              │
        └────────┬──────────────────────┘
                 │
      Tabla: inscripciones (intermedia)
      +----------------+-------------+
      | estudiante_id  | curso_id    |
      +----------------+-------------+
      |              1 |           1 |  ← Ana → Python
      |              1 |           2 |  ← Ana → JavaScript
      |              2 |           1 |  ← Bob → Python
      +----------------+-------------+
```

**SQL:**

```sql
CREATE TABLE estudiantes (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100)
);

CREATE TABLE cursos (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100)
);

CREATE TABLE inscripciones (
    estudiante_id INTEGER,
    curso_id INTEGER,
    PRIMARY KEY (estudiante_id, curso_id),
    FOREIGN KEY (estudiante_id) REFERENCES estudiantes(id),
    FOREIGN KEY (curso_id) REFERENCES cursos(id)
);
```

**Casos comunes:**
- Estudiantes ↔ Cursos
- Productos ↔ Etiquetas
- Actores ↔ Películas

---

### 8.3 Relación Uno a Uno (1:1)

**Descripción:**  
Un registro de una tabla se relaciona con **uno solo** de otra.

```
usuarios (1) ──── (1) perfiles

Un usuario tiene un perfil
Un perfil pertenece a un usuario
```

**Ejemplo:**

```
Tabla: usuarios               Tabla: perfiles
+----+-----------+             +----+------------+-------------+
| id | nombre    |             | id | usuario_id | biografia   |
+----+-----------+             +----+------------+-------------+
|  1 | Ana       | ←────────── |  1 |          1 | Soy Ana...  |
|  2 | Bob       | ←────────── |  2 |          2 | Soy Bob...  |
+----+-----------+             +----+------------+-------------+
```

**SQL:**

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100)
);

CREATE TABLE perfiles (
    id INTEGER PRIMARY KEY,
    usuario_id INTEGER UNIQUE,  -- ← UNIQUE garantiza 1:1
    biografia TEXT,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

**Casos comunes:**
- Usuario → Perfil detallado
- Empleado → Información médica
- Producto → Ficha técnica

---

## 9. Restricciones (Constraints)

Las **restricciones** son reglas que garantizan integridad.

### PRIMARY KEY

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY  -- ← único, no nulo, inmutable
);
```

### FOREIGN KEY

```sql
CREATE TABLE posts (
    usuario_id INTEGER,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

### UNIQUE

```sql
CREATE TABLE usuarios (
    email VARCHAR(255) UNIQUE  -- ← no se repite
);
```

### NOT NULL

```sql
CREATE TABLE usuarios (
    nombre VARCHAR(100) NOT NULL  -- ← obligatorio
);
```

### CHECK

```sql
CREATE TABLE productos (
    precio DECIMAL(10,2) CHECK (precio > 0),  -- ← debe ser positivo
    stock INTEGER CHECK (stock >= 0)          -- ← no negativo
);
```

### DEFAULT

```sql
CREATE TABLE usuarios (
    activo BOOLEAN DEFAULT true,  -- ← valor por defecto
    fecha_registro DATE DEFAULT CURRENT_DATE
);
```

---

## 10. Integridad Referencial

**Definición:**  
Garantiza que las **relaciones sean válidas**.

```
usuarios
+----+-----------+
| id | nombre    |
+----+-----------+
|  1 | Ana       |
+----+-----------+

posts
+----+------------+-----------+
| id | usuario_id | titulo    |
+----+------------+-----------+
|  1 |          1 | Post 1    |  ← válido (usuario 1 existe)
+----+------------+-----------+
```

**Si intentas:**

```sql
-- ❌ ERROR: usuario 999 no existe
INSERT INTO posts (usuario_id, titulo) VALUES (999, 'Post');

-- ❌ ERROR: no puedes eliminar usuario con posts
DELETE FROM usuarios WHERE id = 1;
```

**Opciones con ON DELETE:**

```sql
CREATE TABLE posts (
    usuario_id INTEGER,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
        ON DELETE CASCADE        -- Elimina posts si eliminas usuario
        -- ON DELETE SET NULL    -- Pone NULL en usuario_id
        -- ON DELETE RESTRICT    -- Impide eliminar si hay posts (default)
);
```

---

## 11. Índices (Indexes)

**Definición:**  
Estructuras que aceleran las búsquedas.

**Analogía:**  
Como el índice de un libro (buscar por página sin leer todo).

```sql
-- Sin índice: buscar en toda la tabla
SELECT * FROM usuarios WHERE email = 'ana@example.com';  -- lento

-- Con índice: búsqueda directa
CREATE INDEX idx_email ON usuarios(email);
SELECT * FROM usuarios WHERE email = 'ana@example.com';  -- rápido
```

**Ventajas:**
- ✅ Búsquedas más rápidas
- ✅ Ordenamiento más eficiente

**Desventajas:**
- ❌ Ocupan espacio
- ❌ Ralentizan INSERT/UPDATE

**Cuándo usar:**
- ✅ Columnas que se buscan frecuentemente
- ✅ Claves foráneas
- ✅ Columnas usadas en ORDER BY

---

## 12. Cardinalidad

**Definición:**  
Cantidad de registros relacionados.

### Cardinalidad de relación

```
usuarios (1) ──── (N) posts
         ↓         ↓
      uno       muchos
```

**Tipos:**
- **1:1** (uno a uno)
- **1:N** (uno a muchos)
- **N:M** (muchos a muchos)

### Cardinalidad de columna

```sql
-- Alta cardinalidad (muchos valores únicos)
email: ana@example.com, bob@example.com, ...  ← cada uno diferente

-- Baja cardinalidad (pocos valores repetidos)
genero: M, F, M, M, F, M, F  ← solo 2 valores
```

**Impacto en índices:**
- Alta cardinalidad → buenos índices
- Baja cardinalidad → índices menos útiles

---

## 13. NULL

**Definición:**  
Ausencia de valor (no es 0, no es '', es **nada**).

```sql
INSERT INTO usuarios (nombre, edad) VALUES ('Ana', NULL);
-- edad no se conoce (diferente de edad = 0)
```

**Comportamiento:**

```sql
-- NULL no es igual a nada (ni a sí mismo)
SELECT * FROM usuarios WHERE edad = NULL;    -- ❌ no funciona
SELECT * FROM usuarios WHERE edad IS NULL;   -- ✅ correcto

-- NULL en operaciones
NULL + 10 = NULL
NULL > 5  = NULL (no true, no false)
```

**NOT NULL:**

```sql
CREATE TABLE usuarios (
    nombre VARCHAR(100) NOT NULL  -- obligatorio, no puede ser NULL
);
```

---

## 14. Ejemplo Completo

### E-commerce básico

```sql
-- Tabla: clientes
CREATE TABLE clientes (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    fecha_registro DATE DEFAULT CURRENT_DATE
);

-- Tabla: productos
CREATE TABLE productos (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL,
    precio DECIMAL(10,2) CHECK (precio > 0),
    stock INTEGER CHECK (stock >= 0) DEFAULT 0
);

-- Tabla: pedidos (1:N con clientes)
CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente_id INTEGER NOT NULL,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total DECIMAL(10,2),
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

-- Tabla: items_pedido (N:M entre pedidos y productos)
CREATE TABLE items_pedido (
    pedido_id INTEGER,
    producto_id INTEGER,
    cantidad INTEGER CHECK (cantidad > 0),
    precio_unitario DECIMAL(10,2),
    PRIMARY KEY (pedido_id, producto_id),
    FOREIGN KEY (pedido_id) REFERENCES pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES productos(id)
);

-- Índices
CREATE INDEX idx_cliente ON pedidos(cliente_id);
CREATE INDEX idx_email ON clientes(email);
```

**Relaciones:**
```
clientes (1) ──── (N) pedidos
                       │
                       │ (N)
                       │
                       ▼
                  items_pedido ──── (N) productos
                                    (M)
```

---

## 15. Resumen de Vocabulario

| Término | Significado | Ejemplo |
|---------|-------------|---------|
| **Tabla** | Estructura de datos | `usuarios` |
| **Fila/Registro** | Dato individual | Un usuario |
| **Columna/Campo** | Atributo | `nombre`, `email` |
| **Esquema** | Definición de tabla | `CREATE TABLE` |
| **PRIMARY KEY** | ID único | `id` |
| **FOREIGN KEY** | Referencia a otra tabla | `usuario_id` |
| **Relación 1:N** | Uno a muchos | Usuario → Posts |
| **Relación N:M** | Muchos a muchos | Estudiantes ↔ Cursos |
| **Relación 1:1** | Uno a uno | Usuario → Perfil |
| **UNIQUE** | Sin duplicados | `email` |
| **NOT NULL** | Obligatorio | `nombre` |
| **DEFAULT** | Valor por defecto | `activo = true` |
| **CHECK** | Validación | `precio > 0` |
| **Índice** | Acelera búsquedas | `CREATE INDEX` |
| **NULL** | Ausencia de valor | `WHERE x IS NULL` |

---

## 16. Ejercicios Conceptuales

### Ejercicio 1: Identifica relaciones

**Sistema de blog:**
- Un **usuario** puede escribir muchos **posts**
- Un **post** tiene muchos **comentarios**
- Un **usuario** puede hacer muchos **comentarios**
- Un **post** pertenece a una **categoría**
- Una **categoría** tiene muchos **posts**

**Respuesta:**
```
usuarios (1) ──── (N) posts
usuarios (1) ──── (N) comentarios
posts (1) ──── (N) comentarios
categorias (1) ──── (N) posts
```

---

### Ejercicio 2: Diseña esquema

**Biblioteca:**
- **Libros** (título, ISBN, año)
- **Autores** (nombre, nacionalidad)
- **Préstamos** (fecha_prestamo, fecha_devolucion)
- **Usuarios** (nombre, email)

**Relaciones:**
- Un libro puede tener varios autores
- Un autor puede escribir varios libros
- Un usuario puede tener varios préstamos
- Un libro puede ser prestado varias veces

**Respuesta:**

```sql
CREATE TABLE autores (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100),
    nacionalidad VARCHAR(50)
);

CREATE TABLE libros (
    id INTEGER PRIMARY KEY,
    titulo VARCHAR(200),
    isbn VARCHAR(13) UNIQUE,
    anio INTEGER
);

CREATE TABLE libros_autores (  -- N:M
    libro_id INTEGER,
    autor_id INTEGER,
    PRIMARY KEY (libro_id, autor_id),
    FOREIGN KEY (libro_id) REFERENCES libros(id),
    FOREIGN KEY (autor_id) REFERENCES autores(id)
);

CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(255) UNIQUE
);

CREATE TABLE prestamos (
    id INTEGER PRIMARY KEY,
    libro_id INTEGER,
    usuario_id INTEGER,
    fecha_prestamo DATE,
    fecha_devolucion DATE,
    FOREIGN KEY (libro_id) REFERENCES libros(id),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

---

## Resumen

**Conceptos clave:**
- ✅ **Tabla:** estructura de datos
- ✅ **Fila:** registro individual
- ✅ **Columna:** atributo
- ✅ **PRIMARY KEY:** identificador único
- ✅ **FOREIGN KEY:** referencia a otra tabla
- ✅ **Relaciones:** 1:1, 1:N, N:M
- ✅ **Restricciones:** UNIQUE, NOT NULL, CHECK
- ✅ **Índices:** aceleran búsquedas
- ✅ **NULL:** ausencia de valor

**Próximo módulo:** Normalización (cómo organizar datos correctamente)

---

## Recursos

- **[PostgreSQL Data Types](https://www.postgresql.org/docs/current/datatype.html)** - Tipos de datos
- **[Database Keys](https://www.w3schools.com/sql/sql_foreignkey.asp)** - Claves primarias y foráneas
- **[Database Relationships](https://www.lucidchart.com/pages/database-diagram/database-design)** - Relaciones visuales

Ahora dominas los **conceptos fundamentales** de bases de datos. 📊

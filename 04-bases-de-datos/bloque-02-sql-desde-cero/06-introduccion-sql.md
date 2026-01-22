# Introducción a SQL

> **El lenguaje universal de las bases de datos relacionales**

---

## ¿Qué es SQL?

**SQL** significa **Structured Query Language** (Lenguaje de Consulta Estructurado).

**Definición:**  
SQL es el lenguaje estándar para:
- 📖 **Consultar** datos (leer)
- ✏️ **Modificar** datos (crear, actualizar, eliminar)
- 🏗️ **Definir** estructura (tablas, columnas)
- 🔐 **Controlar** acceso (permisos)

**Analogía:**  
SQL es como el "idioma" que usas para hablar con la base de datos.

```
Tú: "Dame todos los usuarios mayores de 25 años"
BD: [lista de usuarios]
```

---

## Historia rápida

- **1970:** Edgar F. Codd inventa modelo relacional
- **1974:** IBM crea SEQUEL (después SQL)
- **1986:** SQL se convierte en estándar ANSI
- **Hoy:** Usado por PostgreSQL, MySQL, Oracle, SQL Server, SQLite

---

## Características de SQL

### ✅ Lenguaje declarativo

**No dices "cómo", dices "qué"**

```python
# Python (imperativo): dices CÓMO
for usuario in usuarios:
    if usuario.edad > 25:
        print(usuario.nombre)

# SQL (declarativo): dices QUÉ
SELECT nombre FROM usuarios WHERE edad > 25;
```

### ✅ Estándar universal

```sql
-- Funciona en PostgreSQL, MySQL, SQLite, Oracle...
SELECT * FROM usuarios WHERE edad > 25;
```

### ✅ No sensible a mayúsculas (palabras clave)

```sql
-- Todos válidos
SELECT * FROM usuarios;
select * from usuarios;
SeLeCt * FrOm usuarios;  -- no recomendado
```

**Convención:** Palabras clave en MAYÚSCULAS, nombres en minúsculas.

### ✅ Termina con punto y coma

```sql
SELECT * FROM usuarios;  -- ← punto y coma
```

---

## Categorías de SQL

SQL se divide en **subcategorías**:

```
SQL
├── DDL (Data Definition Language)
│   └── Definir estructura (CREATE, ALTER, DROP)
│
├── DML (Data Manipulation Language)
│   └── Manipular datos (INSERT, UPDATE, DELETE)
│
├── DQL (Data Query Language)
│   └── Consultar datos (SELECT)
│
├── DCL (Data Control Language)
│   └── Control de acceso (GRANT, REVOKE)
│
└── TCL (Transaction Control Language)
    └── Transacciones (BEGIN, COMMIT, ROLLBACK)
```

**En este curso nos enfocamos en:**
- ✅ **DDL** (crear tablas)
- ✅ **DML** (insertar, actualizar, eliminar)
- ✅ **DQL** (consultas - lo más importante)

---

## Tu primera consulta SQL

### Escenario: Tabla de usuarios

```
Tabla: usuarios
+----+-----------+-------------------+------+
| id | nombre    | email             | edad |
+----+-----------+-------------------+------+
|  1 | Ana       | ana@example.com   |   25 |
|  2 | Bob       | bob@example.com   |   30 |
|  3 | Carlos    | carlos@example.com|   28 |
+----+-----------+-------------------+------+
```

### Consulta 1: Ver todos los datos

```sql
SELECT * FROM usuarios;
```

**Resultado:**
```
+----+-----------+-------------------+------+
| id | nombre    | email             | edad |
+----+-----------+-------------------+------+
|  1 | Ana       | ana@example.com   |   25 |
|  2 | Bob       | bob@example.com   |   30 |
|  3 | Carlos    | carlos@example.com|   28 |
+----+-----------+-------------------+------+
```

**Desglose:**
- `SELECT`: palabra clave para consultar
- `*`: todos los campos
- `FROM`: de qué tabla
- `usuarios`: nombre de la tabla

---

### Consulta 2: Columnas específicas

```sql
SELECT nombre, edad FROM usuarios;
```

**Resultado:**
```
+-----------+------+
| nombre    | edad |
+-----------+------+
| Ana       |   25 |
| Bob       |   30 |
| Carlos    |   28 |
+-----------+------+
```

---

### Consulta 3: Con filtro

```sql
SELECT nombre, edad 
FROM usuarios 
WHERE edad > 25;
```

**Resultado:**
```
+-----------+------+
| nombre    | edad |
+-----------+------+
| Bob       |   30 |
| Carlos    |   28 |
+-----------+------+
```

---

### Consulta 4: Ordenado

```sql
SELECT nombre, edad 
FROM usuarios 
ORDER BY edad DESC;
```

**Resultado:**
```
+-----------+------+
| nombre    | edad |
+-----------+------+
| Bob       |   30 |
| Carlos    |   28 |
| Ana       |   25 |
+-----------+------+
```

---

## Anatomía de una consulta SELECT

```sql
SELECT columna1, columna2          -- 1. Qué campos quieres
FROM tabla                         -- 2. De qué tabla
WHERE condicion                    -- 3. Filtrar filas (opcional)
ORDER BY columna ASC/DESC          -- 4. Ordenar (opcional)
LIMIT 10;                          -- 5. Limitar resultados (opcional)
```

**Orden de ejecución (interno):**
1. `FROM` - base de datos identifica la tabla
2. `WHERE` - filtra las filas
3. `SELECT` - selecciona columnas
4. `ORDER BY` - ordena resultados
5. `LIMIT` - limita cantidad

---

## Tipos de datos en SQL

### Numéricos

```sql
INTEGER         -- Enteros: 1, 2, 100
BIGINT          -- Enteros grandes
DECIMAL(10,2)   -- Decimales exactos: 19.99
FLOAT           -- Decimales aproximados
REAL            -- Decimales de precisión simple
```

### Texto

```sql
CHAR(5)         -- Texto fijo: 'HOLA '
VARCHAR(100)    -- Texto variable: 'Hola' (usa 4 bytes)
TEXT            -- Texto largo sin límite
```

### Fecha y hora

```sql
DATE            -- Fecha: 2024-01-01
TIME            -- Hora: 14:30:00
DATETIME        -- Fecha y hora: 2024-01-01 14:30:00
TIMESTAMP       -- Con zona horaria
```

### Booleano

```sql
BOOLEAN         -- true/false (1/0)
```

---

## SQLite: Nuestra herramienta

**¿Por qué SQLite?**

✅ **No requiere instalación** (archivo único)  
✅ **Liviano** (ideal para aprender)  
✅ **SQL estándar** (lo que aprendes aplica a PostgreSQL, MySQL)  
✅ **Usado en producción** (apps móviles, navegadores)

### Instalar SQLite

**Windows:**
1. Descargar desde https://www.sqlite.org/download.html
2. Buscar "sqlite-tools-win32"
3. Descomprimir y agregar al PATH

**macOS:**
```bash
# Ya viene instalado
sqlite3 --version
```

**Linux:**
```bash
sudo apt install sqlite3
```

### Verificar instalación

```bash
sqlite3 --version
# SQLite 3.x.x
```

---

## Primer contacto con SQLite

### Crear base de datos

```bash
sqlite3 mi_base_datos.db
```

Esto abre la consola SQLite:
```
SQLite version 3.x.x
Enter ".help" for usage hints.
sqlite>
```

### Comandos especiales de SQLite

```sql
.help                  -- Ayuda
.tables                -- Ver tablas
.schema nombre_tabla   -- Ver estructura de tabla
.quit                  -- Salir
.mode column           -- Formato de columnas
.headers on            -- Mostrar encabezados
```

---

## Ejemplo práctico completo

### Paso 1: Crear base de datos

```bash
sqlite3 usuarios.db
```

### Paso 2: Configurar formato

```sql
.mode column
.headers on
```

### Paso 3: Crear tabla

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    edad INTEGER
);
```

### Paso 4: Insertar datos

```sql
INSERT INTO usuarios (id, nombre, email, edad)
VALUES (1, 'Ana García', 'ana@example.com', 25);

INSERT INTO usuarios (id, nombre, email, edad)
VALUES (2, 'Bob Smith', 'bob@example.com', 30);

INSERT INTO usuarios (id, nombre, email, edad)
VALUES (3, 'Carlos Ruiz', 'carlos@example.com', 28);
```

### Paso 5: Consultar

```sql
-- Todos los usuarios
SELECT * FROM usuarios;

-- Solo nombres y edades
SELECT nombre, edad FROM usuarios;

-- Usuarios mayores de 25
SELECT nombre, edad FROM usuarios WHERE edad > 25;

-- Ordenados por edad descendente
SELECT nombre, edad FROM usuarios ORDER BY edad DESC;
```

### Paso 6: Salir

```sql
.quit
```

---

## Operadores comunes

### Comparación

```sql
=       -- Igual a
!=      -- Diferente de
<>      -- Diferente de (alternativa)
>       -- Mayor que
<       -- Menor que
>=      -- Mayor o igual
<=      -- Menor o igual
```

**Ejemplos:**

```sql
SELECT * FROM usuarios WHERE edad = 25;
SELECT * FROM usuarios WHERE edad > 25;
SELECT * FROM usuarios WHERE edad >= 25;
SELECT * FROM usuarios WHERE edad != 30;
```

---

### Lógicos

```sql
AND     -- Y (ambas condiciones)
OR      -- O (cualquier condición)
NOT     -- Negación
```

**Ejemplos:**

```sql
-- Usuarios entre 25 y 30 años
SELECT * FROM usuarios WHERE edad >= 25 AND edad <= 30;

-- Usuarios que NO son Carlos
SELECT * FROM usuarios WHERE NOT nombre = 'Carlos';

-- Usuarios con edad 25 o 30
SELECT * FROM usuarios WHERE edad = 25 OR edad = 30;
```

---

### Patrones (LIKE)

```sql
LIKE    -- Coincidencia de patrones
%       -- Cualquier secuencia de caracteres
_       -- Un solo carácter
```

**Ejemplos:**

```sql
-- Nombres que empiezan con 'A'
SELECT * FROM usuarios WHERE nombre LIKE 'A%';

-- Emails de gmail
SELECT * FROM usuarios WHERE email LIKE '%@gmail.com';

-- Nombres con 3 letras
SELECT * FROM usuarios WHERE nombre LIKE '___';

-- Nombres que contienen 'ar'
SELECT * FROM usuarios WHERE nombre LIKE '%ar%';
```

---

### Rango (BETWEEN)

```sql
BETWEEN valor1 AND valor2
```

**Ejemplos:**

```sql
-- Usuarios entre 25 y 30 años
SELECT * FROM usuarios WHERE edad BETWEEN 25 AND 30;

-- Equivalente a:
SELECT * FROM usuarios WHERE edad >= 25 AND edad <= 30;
```

---

### Lista (IN)

```sql
IN (valor1, valor2, ...)
```

**Ejemplos:**

```sql
-- Usuarios con edad 25, 28 o 30
SELECT * FROM usuarios WHERE edad IN (25, 28, 30);

-- Equivalente a:
SELECT * FROM usuarios 
WHERE edad = 25 OR edad = 28 OR edad = 30;
```

---

### Nulos (IS NULL)

```sql
IS NULL
IS NOT NULL
```

**Ejemplos:**

```sql
-- Usuarios sin email
SELECT * FROM usuarios WHERE email IS NULL;

-- Usuarios con email
SELECT * FROM usuarios WHERE email IS NOT NULL;

-- ❌ NO funciona
SELECT * FROM usuarios WHERE email = NULL;  -- siempre falso
```

---

## Alias

Renombrar columnas o tablas temporalmente.

```sql
-- Alias de columna
SELECT nombre AS usuario_nombre, edad AS usuario_edad
FROM usuarios;

-- Resultado:
-- +-----------------+--------------+
-- | usuario_nombre  | usuario_edad |
-- +-----------------+--------------+
-- | Ana             |           25 |
-- +-----------------+--------------+

-- Alias de tabla
SELECT u.nombre, u.edad
FROM usuarios AS u;

-- Sin AS (también válido)
SELECT u.nombre, u.edad
FROM usuarios u;
```

---

## Funciones básicas

### Contar filas

```sql
SELECT COUNT(*) FROM usuarios;
-- Resultado: 3
```

### Calcular promedio

```sql
SELECT AVG(edad) FROM usuarios;
-- Resultado: 27.666...
```

### Mínimo y máximo

```sql
SELECT MIN(edad) FROM usuarios;
-- Resultado: 25

SELECT MAX(edad) FROM usuarios;
-- Resultado: 30
```

### Suma

```sql
SELECT SUM(edad) FROM usuarios;
-- Resultado: 83
```

---

## Comentarios en SQL

```sql
-- Comentario de una línea

/* Comentario
   de múltiples
   líneas */

SELECT * FROM usuarios;  -- comentario al final
```

---

## Buenas prácticas

### ✅ Mayúsculas para palabras clave

```sql
-- BIEN
SELECT nombre FROM usuarios WHERE edad > 25;

-- Evitar
select nombre from usuarios where edad > 25;
```

### ✅ Indentación y saltos de línea

```sql
-- BIEN
SELECT 
    nombre, 
    email, 
    edad
FROM usuarios
WHERE edad > 25
ORDER BY nombre;

-- Evitar (difícil de leer)
SELECT nombre,email,edad FROM usuarios WHERE edad>25 ORDER BY nombre;
```

### ✅ Nombres descriptivos

```sql
-- BIEN
SELECT nombre, email FROM usuarios;

-- Evitar
SELECT n, e FROM u;
```

### ✅ Usar alias para claridad

```sql
-- BIEN
SELECT 
    u.nombre AS usuario_nombre,
    u.edad AS usuario_edad
FROM usuarios AS u;
```

---

## Errores comunes

### ❌ Olvidar punto y coma

```sql
SELECT * FROM usuarios  -- ❌ sin ;
```

### ❌ Comparar NULL con =

```sql
SELECT * FROM usuarios WHERE email = NULL;  -- ❌ siempre falso

SELECT * FROM usuarios WHERE email IS NULL; -- ✅ correcto
```

### ❌ Comillas simples para strings

```sql
-- SQL usa comillas simples
SELECT * FROM usuarios WHERE nombre = 'Ana';  -- ✅

SELECT * FROM usuarios WHERE nombre = "Ana";  -- ⚠️ funciona en algunos motores
```

### ❌ Sensibilidad a mayúsculas en nombres

```sql
-- En algunos motores (PostgreSQL)
SELECT * FROM Usuarios;  -- ❌ si la tabla es "usuarios"
SELECT * FROM usuarios;  -- ✅
```

---

## Comparación: Python vs SQL

### Filtrar datos

**Python:**
```python
usuarios = [
    {'nombre': 'Ana', 'edad': 25},
    {'nombre': 'Bob', 'edad': 30},
    {'nombre': 'Carlos', 'edad': 28}
]

mayores_25 = [u for u in usuarios if u['edad'] > 25]
print(mayores_25)
```

**SQL:**
```sql
SELECT * FROM usuarios WHERE edad > 25;
```

### Contar

**Python:**
```python
len(usuarios)
```

**SQL:**
```sql
SELECT COUNT(*) FROM usuarios;
```

### Ordenar

**Python:**
```python
sorted(usuarios, key=lambda u: u['edad'], reverse=True)
```

**SQL:**
```sql
SELECT * FROM usuarios ORDER BY edad DESC;
```

---

## Resumen

**SQL es:**
- 📖 Lenguaje estándar para bases de datos relacionales
- 🗣️ Declarativo (dices QUÉ, no CÓMO)
- 🔧 Universal (funciona en PostgreSQL, MySQL, SQLite...)

**Categorías:**
- **DDL:** Definir estructura (CREATE, ALTER, DROP)
- **DML:** Manipular datos (INSERT, UPDATE, DELETE)
- **DQL:** Consultar (SELECT)

**Consulta básica:**
```sql
SELECT columnas
FROM tabla
WHERE condicion
ORDER BY columna;
```

**SQLite:**
- ✅ Ideal para aprender
- ✅ No requiere servidor
- ✅ SQL estándar

**Próximo módulo:** DDL - Crear y modificar tablas

---

## Ejercicios

### Ejercicio 1

Dado:
```sql
SELECT * FROM productos WHERE precio > 100;
```

¿Qué hace esta consulta?

**Respuesta:** Selecciona todos los productos con precio mayor a 100.

---

### Ejercicio 2

Escribe una consulta para:
- Ver nombre y email de usuarios
- Donde edad sea 25 o 30
- Ordenados por nombre

**Respuesta:**
```sql
SELECT nombre, email
FROM usuarios
WHERE edad IN (25, 30)
ORDER BY nombre;
```

---

### Ejercicio 3

¿Cuál es el error?
```sql
SELECT * FROM usuarios WHERE email = NULL;
```

**Respuesta:** Debe usar `IS NULL`:
```sql
SELECT * FROM usuarios WHERE email IS NULL;
```

---

## Recursos

- **[SQLite Tutorial](https://www.sqlitetutorial.net/)** - Tutorial completo
- **[SQL Playground](https://www.db-fiddle.com/)** - Practicar online
- **[SQL Cheat Sheet](https://www.sqltutorial.org/sql-cheat-sheet/)** - Referencia rápida
- **[SQLBolt](https://sqlbolt.com/)** - Ejercicios interactivos

Ahora conoces **los fundamentos de SQL**. 🚀

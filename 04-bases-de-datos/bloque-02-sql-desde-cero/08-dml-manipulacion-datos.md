# DML - Data Manipulation Language

> **Insertar, actualizar y eliminar datos**

---

## ¿Qué es DML?

**DML** (Data Manipulation Language) son los comandos SQL para **manipular datos** (no estructura):

```
DML
├── INSERT    (Insertar datos)
├── UPDATE    (Actualizar datos)
└── DELETE    (Eliminar datos)
```

**Manipulan filas, no columnas.**

---

## INSERT - Insertar datos

Agregar nuevas filas a una tabla.

### Sintaxis básica

```sql
INSERT INTO tabla (columna1, columna2, ...)
VALUES (valor1, valor2, ...);
```

### Ejemplo simple

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    email TEXT
);

INSERT INTO usuarios (id, nombre, email)
VALUES (1, 'Ana García', 'ana@example.com');
```

---

### Insertar sin especificar columnas

**Si incluyes todas las columnas en orden:**

```sql
INSERT INTO usuarios
VALUES (2, 'Bob Smith', 'bob@example.com');
```

**⚠️ Riesgoso:** Si cambias estructura, se rompe.

**Mejor práctica:** Siempre especificar columnas.

---

### Insertar con columnas opcionales

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    email TEXT NOT NULL,
    telefono TEXT,           -- opcional
    activo INTEGER DEFAULT 1 -- con default
);

-- Solo campos obligatorios
INSERT INTO usuarios (nombre, email)
VALUES ('Carlos', 'carlos@example.com');

-- activo = 1 (default), telefono = NULL
```

---

### Insertar múltiples filas

```sql
INSERT INTO usuarios (nombre, email)
VALUES 
    ('Ana García', 'ana@example.com'),
    ('Bob Smith', 'bob@example.com'),
    ('Carlos Ruiz', 'carlos@example.com');
```

**Más eficiente que INSERT individuales.**

---

### Insertar con valores calculados

```sql
INSERT INTO productos (nombre, precio, precio_con_iva)
VALUES ('Laptop', 1000, 1000 * 1.19);

-- precio_con_iva = 1190
```

---

### Insertar con funciones

```sql
INSERT INTO usuarios (nombre, email, fecha_registro)
VALUES ('Diana', 'diana@example.com', CURRENT_DATE);

-- fecha_registro = fecha actual
```

---

### Insertar desde otra tabla (INSERT SELECT)

```sql
-- Crear tabla backup
CREATE TABLE usuarios_backup (
    id INTEGER,
    nombre TEXT,
    email TEXT
);

-- Copiar usuarios activos
INSERT INTO usuarios_backup
SELECT id, nombre, email
FROM usuarios
WHERE activo = 1;
```

---

### INSERT OR IGNORE

**SQLite:** Ignorar si viola restricción.

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    email TEXT UNIQUE
);

INSERT INTO usuarios (id, email) VALUES (1, 'ana@example.com');

-- Intenta insertar email duplicado
INSERT OR IGNORE INTO usuarios (id, email) 
VALUES (2, 'ana@example.com');

-- No error, simplemente ignora
```

---

### INSERT OR REPLACE

**SQLite:** Reemplazar si existe.

```sql
INSERT OR REPLACE INTO usuarios (id, nombre, email)
VALUES (1, 'Ana García Actualizado', 'ana@example.com');

-- Si id=1 existe, lo reemplaza
-- Si no existe, lo inserta
```

---

## UPDATE - Actualizar datos

Modificar filas existentes.

### Sintaxis básica

```sql
UPDATE tabla
SET columna1 = valor1, columna2 = valor2, ...
WHERE condicion;
```

**⚠️ IMPORTANTE:** Siempre usa WHERE (o actualizas TODAS las filas).

---

### Ejemplo simple

```sql
-- Actualizar email de un usuario
UPDATE usuarios
SET email = 'ana_nuevo@example.com'
WHERE id = 1;
```

---

### Actualizar múltiples columnas

```sql
UPDATE usuarios
SET 
    nombre = 'Ana García Rodríguez',
    email = 'ana_nuevo@example.com',
    telefono = '555-1234'
WHERE id = 1;
```

---

### Actualizar basado en condición

```sql
-- Activar usuarios mayores de 25
UPDATE usuarios
SET activo = 1
WHERE edad > 25;
```

---

### Actualizar con cálculos

```sql
-- Aumentar precio 10%
UPDATE productos
SET precio = precio * 1.10
WHERE categoria = 'Electrónica';
```

---

### Actualizar con funciones

```sql
-- Actualizar fecha de modificación
UPDATE posts
SET fecha_actualizacion = CURRENT_TIMESTAMP
WHERE id = 1;
```

---

### Actualizar todas las filas

**⚠️ PELIGROSO:**

```sql
-- SIN WHERE: actualiza TODAS las filas
UPDATE usuarios
SET activo = 0;

-- Todos los usuarios ahora tienen activo = 0
```

**Mejor práctica:** Siempre confirma con SELECT antes.

```sql
-- 1. Ver qué filas se afectarán
SELECT * FROM usuarios WHERE edad > 25;

-- 2. Si está bien, UPDATE
UPDATE usuarios SET activo = 1 WHERE edad > 25;
```

---

### UPDATE con subconsulta

```sql
-- Actualizar stock basado en ventas
UPDATE productos
SET stock = stock - (
    SELECT SUM(cantidad)
    FROM items_pedido
    WHERE producto_id = productos.id
);
```

---

## DELETE - Eliminar datos

Eliminar filas existentes.

### Sintaxis básica

```sql
DELETE FROM tabla
WHERE condicion;
```

**⚠️ IMPORTANTE:** Siempre usa WHERE (o eliminas TODAS las filas).

---

### Ejemplo simple

```sql
-- Eliminar un usuario
DELETE FROM usuarios
WHERE id = 1;
```

---

### Eliminar basado en condición

```sql
-- Eliminar usuarios inactivos
DELETE FROM usuarios
WHERE activo = 0;
```

---

### Eliminar múltiples filas

```sql
-- Eliminar usuarios mayores de 60 años
DELETE FROM usuarios
WHERE edad > 60;
```

---

### Eliminar con subconsulta

```sql
-- Eliminar productos sin stock
DELETE FROM productos
WHERE id NOT IN (
    SELECT DISTINCT producto_id
    FROM items_pedido
);
```

---

### Eliminar todas las filas

**⚠️ PELIGROSO:**

```sql
-- SIN WHERE: elimina TODAS las filas
DELETE FROM usuarios;

-- Tabla vacía (pero estructura existe)
```

**Mejor práctica:** Usa transacciones o confirma con SELECT.

```sql
-- 1. Ver qué filas se eliminarán
SELECT * FROM usuarios WHERE activo = 0;

-- 2. Si está bien, DELETE
DELETE FROM usuarios WHERE activo = 0;
```

---

## Ejemplo práctico completo

### Crear base de datos de productos

```sql
sqlite3 tienda.db

.mode column
.headers on

-- Crear tabla
CREATE TABLE productos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    precio REAL CHECK (precio > 0),
    stock INTEGER CHECK (stock >= 0) DEFAULT 0,
    categoria TEXT,
    activo INTEGER DEFAULT 1
);
```

---

### INSERT: Agregar productos

```sql
-- Un producto
INSERT INTO productos (nombre, precio, stock, categoria)
VALUES ('Laptop Dell', 1200, 10, 'Electrónica');

-- Múltiples productos
INSERT INTO productos (nombre, precio, stock, categoria)
VALUES 
    ('Mouse Logitech', 25, 50, 'Accesorios'),
    ('Teclado Mecánico', 80, 30, 'Accesorios'),
    ('Monitor LG 24"', 200, 15, 'Electrónica'),
    ('Webcam HD', 50, 20, 'Accesorios');

-- Ver datos
SELECT * FROM productos;
```

**Resultado:**
```
+----+-------------------+-------+-------+--------------+--------+
| id | nombre            | precio| stock | categoria    | activo |
+----+-------------------+-------+-------+--------------+--------+
|  1 | Laptop Dell       | 1200  |    10 | Electrónica  |      1 |
|  2 | Mouse Logitech    |   25  |    50 | Accesorios   |      1 |
|  3 | Teclado Mecánico  |   80  |    30 | Accesorios   |      1 |
|  4 | Monitor LG 24"    |  200  |    15 | Electrónica  |      1 |
|  5 | Webcam HD         |   50  |    20 | Accesorios   |      1 |
+----+-------------------+-------+-------+--------------+--------+
```

---

### UPDATE: Actualizar productos

```sql
-- Aumentar precio de electrónica 10%
UPDATE productos
SET precio = precio * 1.10
WHERE categoria = 'Electrónica';

-- Reducir stock de Mouse (venta)
UPDATE productos
SET stock = stock - 5
WHERE id = 2;

-- Cambiar nombre
UPDATE productos
SET nombre = 'Mouse Logitech MX Master'
WHERE id = 2;

-- Ver cambios
SELECT * FROM productos;
```

**Resultado:**
```
+----+---------------------------+-------+-------+--------------+--------+
| id | nombre                    | precio| stock | categoria    | activo |
+----+---------------------------+-------+-------+--------------+--------+
|  1 | Laptop Dell               | 1320  |    10 | Electrónica  |      1 |
|  2 | Mouse Logitech MX Master  |   25  |    45 | Accesorios   |      1 |
|  3 | Teclado Mecánico          |   80  |    30 | Accesorios   |      1 |
|  4 | Monitor LG 24"            |  220  |    15 | Electrónica  |      1 |
|  5 | Webcam HD                 |   50  |    20 | Accesorios   |      1 |
+----+---------------------------+-------+-------+--------------+--------+
```

---

### DELETE: Eliminar productos

```sql
-- Eliminar productos sin stock
DELETE FROM productos
WHERE stock = 0;

-- Desactivar en lugar de eliminar (soft delete)
UPDATE productos
SET activo = 0
WHERE id = 5;

-- Ver solo activos
SELECT * FROM productos WHERE activo = 1;
```

---

## Transacciones

Agrupar operaciones (todo o nada).

### Sintaxis

```sql
BEGIN TRANSACTION;

-- operaciones DML

COMMIT;   -- confirmar
-- o
ROLLBACK; -- revertir
```

---

### Ejemplo: Transferencia de stock

```sql
BEGIN TRANSACTION;

-- Reducir stock del almacén A
UPDATE productos 
SET stock = stock - 10
WHERE id = 1 AND stock >= 10;

-- Aumentar stock del almacén B
UPDATE productos_almacen_b
SET stock = stock + 10
WHERE id = 1;

-- Si ambos funcionan, confirmar
COMMIT;

-- Si algo falla, revertir
-- ROLLBACK;
```

---

### Ejemplo: Crear pedido

```sql
BEGIN TRANSACTION;

-- 1. Insertar pedido
INSERT INTO pedidos (cliente_id, total)
VALUES (1, 1220);

-- Obtener id del pedido
-- En SQLite: last_insert_rowid()

-- 2. Insertar items
INSERT INTO items_pedido (pedido_id, producto_id, cantidad, precio_unitario)
VALUES (last_insert_rowid(), 1, 1, 1200);

INSERT INTO items_pedido (pedido_id, producto_id, cantidad, precio_unitario)
VALUES (last_insert_rowid(), 2, 1, 20);

-- 3. Reducir stock
UPDATE productos SET stock = stock - 1 WHERE id = 1;
UPDATE productos SET stock = stock - 1 WHERE id = 2;

-- Todo bien, confirmar
COMMIT;
```

---

## RETURNING (PostgreSQL/SQLite 3.35+)

Devolver datos insertados/actualizados.

```sql
-- Insertar y devolver id
INSERT INTO usuarios (nombre, email)
VALUES ('Eva', 'eva@example.com')
RETURNING id, nombre;

-- Resultado:
-- +----+------+
-- | id | nombre|
-- +----+------+
-- |  6 | Eva  |
-- +----+------+
```

**SQLite < 3.35:** Usa `last_insert_rowid()`.

```sql
INSERT INTO usuarios (nombre, email)
VALUES ('Eva', 'eva@example.com');

SELECT last_insert_rowid();
```

---

## Operaciones masivas

### Insertar desde CSV (SQLite)

```sql
.mode csv
.import datos.csv usuarios
```

**datos.csv:**
```
nombre,email,edad
Ana,ana@example.com,25
Bob,bob@example.com,30
```

---

### Exportar a CSV

```sql
.mode csv
.output usuarios.csv
SELECT * FROM usuarios;
.output stdout
```

---

## Soft Delete (Eliminación lógica)

**Mejor práctica:** No eliminar, desactivar.

```sql
-- Agregar columna
ALTER TABLE usuarios
ADD COLUMN eliminado INTEGER DEFAULT 0;

-- En lugar de DELETE
-- DELETE FROM usuarios WHERE id = 1;

-- Hacer UPDATE
UPDATE usuarios
SET eliminado = 1
WHERE id = 1;

-- Consultar solo no eliminados
SELECT * FROM usuarios WHERE eliminado = 0;
```

**Ventajas:**
- ✅ Puedes recuperar datos
- ✅ Auditoría
- ✅ No rompes relaciones

---

## Buenas prácticas

### ✅ Siempre WHERE en UPDATE/DELETE

```sql
-- MAL
UPDATE usuarios SET activo = 0;  -- actualiza TODO

-- BIEN
UPDATE usuarios SET activo = 0 WHERE id = 1;
```

### ✅ Confirmar con SELECT antes

```sql
-- 1. Ver qué se afectará
SELECT * FROM usuarios WHERE edad > 60;

-- 2. Si está bien, ejecutar
DELETE FROM usuarios WHERE edad > 60;
```

### ✅ Usar transacciones para operaciones múltiples

```sql
BEGIN TRANSACTION;
-- operaciones
COMMIT;
```

### ✅ Usar soft delete

```sql
-- No DELETE
UPDATE usuarios SET eliminado = 1 WHERE id = 1;
```

### ✅ Validar antes de insertar

```sql
-- Verificar que no exista
SELECT COUNT(*) FROM usuarios WHERE email = 'ana@example.com';

-- Si 0, insertar
INSERT INTO usuarios (nombre, email) 
VALUES ('Ana', 'ana@example.com');
```

---

## Errores comunes

### ❌ UPDATE/DELETE sin WHERE

```sql
-- ❌ PELIGROSO
DELETE FROM usuarios;  -- elimina TODO

-- ✅ SEGURO
DELETE FROM usuarios WHERE id = 1;
```

### ❌ No validar restricciones

```sql
-- ❌ ERROR: email duplicado
INSERT INTO usuarios (email) VALUES ('ana@example.com');
INSERT INTO usuarios (email) VALUES ('ana@example.com');

-- ✅ BIEN: verificar antes
SELECT COUNT(*) FROM usuarios WHERE email = 'ana@example.com';
-- Si 0, insertar
```

### ❌ No usar transacciones

```sql
-- MAL: si falla UPDATE 2, UPDATE 1 queda inconsistente
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;

-- BIEN: todo o nada
BEGIN TRANSACTION;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
COMMIT;
```

---

## Comparación: Python vs SQL

### Insertar

**Python:**
```python
usuario = {'nombre': 'Ana', 'email': 'ana@example.com'}
usuarios.append(usuario)
```

**SQL:**
```sql
INSERT INTO usuarios (nombre, email)
VALUES ('Ana', 'ana@example.com');
```

---

### Actualizar

**Python:**
```python
for usuario in usuarios:
    if usuario['id'] == 1:
        usuario['email'] = 'ana_nuevo@example.com'
```

**SQL:**
```sql
UPDATE usuarios
SET email = 'ana_nuevo@example.com'
WHERE id = 1;
```

---

### Eliminar

**Python:**
```python
usuarios = [u for u in usuarios if u['id'] != 1]
```

**SQL:**
```sql
DELETE FROM usuarios WHERE id = 1;
```

---

## Resumen

**DML:**
- `INSERT` - Agregar filas
- `UPDATE` - Modificar filas
- `DELETE` - Eliminar filas

**INSERT:**
```sql
INSERT INTO tabla (col1, col2) VALUES (val1, val2);
```

**UPDATE:**
```sql
UPDATE tabla SET col1 = val1 WHERE condicion;
```

**DELETE:**
```sql
DELETE FROM tabla WHERE condicion;
```

**Transacciones:**
```sql
BEGIN TRANSACTION;
-- operaciones
COMMIT;
```

**Soft Delete:**
```sql
UPDATE tabla SET eliminado = 1 WHERE id = 1;
```

**Próximo módulo:** Filtrado y ordenamiento (WHERE, ORDER BY)

---

## Ejercicios

### Ejercicio 1

Insertar 3 productos:
- Laptop (1000, stock 10)
- Mouse (20, stock 50)
- Teclado (80, stock 30)

**Solución:**
```sql
INSERT INTO productos (nombre, precio, stock)
VALUES 
    ('Laptop', 1000, 10),
    ('Mouse', 20, 50),
    ('Teclado', 80, 30);
```

---

### Ejercicio 2

Aumentar precio 15% a productos con stock < 20.

**Solución:**
```sql
UPDATE productos
SET precio = precio * 1.15
WHERE stock < 20;
```

---

### Ejercicio 3

Eliminar productos con precio < 10.

**Solución:**
```sql
DELETE FROM productos
WHERE precio < 10;
```

---

### Ejercicio 4

Crear transacción que:
1. Inserte un pedido
2. Inserte items del pedido
3. Reduzca stock

**Solución:**
```sql
BEGIN TRANSACTION;

INSERT INTO pedidos (cliente_id, total) VALUES (1, 100);

INSERT INTO items_pedido (pedido_id, producto_id, cantidad)
VALUES (last_insert_rowid(), 1, 1);

UPDATE productos SET stock = stock - 1 WHERE id = 1;

COMMIT;
```

---

## Recursos

- **[INSERT](https://www.sqlite.org/lang_insert.html)** - Documentación SQLite
- **[UPDATE](https://www.sqlite.org/lang_update.html)** - Documentación SQLite
- **[DELETE](https://www.sqlite.org/lang_delete.html)** - Documentación SQLite
- **[Transactions](https://www.sqlite.org/lang_transaction.html)** - Transacciones

Ahora sabes **manipular datos** en bases de datos. ✏️

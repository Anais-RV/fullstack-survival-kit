# Filtrado y Ordenamiento en SQL

> **WHERE, ORDER BY, LIMIT - Consultas precisas y organizadas**

---

## WHERE - Filtrar filas

El comando `WHERE` filtra filas basándose en condiciones.

### Sintaxis

```sql
SELECT columnas
FROM tabla
WHERE condicion;
```

---

## Operadores de comparación

### Igualdad (=)

```sql
-- Usuarios con edad 25
SELECT * FROM usuarios WHERE edad = 25;
```

### Diferente (!= o <>)

```sql
-- Usuarios que NO tienen 25 años
SELECT * FROM usuarios WHERE edad != 25;
SELECT * FROM usuarios WHERE edad <> 25;  -- equivalente
```

### Mayor que (>)

```sql
-- Usuarios mayores de 25
SELECT * FROM usuarios WHERE edad > 25;
```

### Menor que (<)

```sql
-- Usuarios menores de 25
SELECT * FROM usuarios WHERE edad < 25;
```

### Mayor o igual (>=)

```sql
-- Usuarios con 25 o más años
SELECT * FROM usuarios WHERE edad >= 25;
```

### Menor o igual (<=)

```sql
-- Usuarios con 25 o menos años
SELECT * FROM usuarios WHERE edad <= 25;
```

---

## Operadores lógicos

### AND (Y)

**Ambas condiciones deben cumplirse.**

```sql
-- Usuarios con edad entre 25 y 30
SELECT * FROM usuarios
WHERE edad >= 25 AND edad <= 30;
```

**Ejemplo con tabla:**
```
+----+---------+------+
| id | nombre  | edad |
+----+---------+------+
|  1 | Ana     |   25 | ✅
|  2 | Bob     |   30 | ✅
|  3 | Carlos  |   35 | ❌ (edad > 30)
|  4 | Diana   |   20 | ❌ (edad < 25)
+----+---------+------+
```

---

### OR (O)

**Al menos una condición debe cumplirse.**

```sql
-- Usuarios con edad 25 o 30
SELECT * FROM usuarios
WHERE edad = 25 OR edad = 30;
```

**Ejemplo:**
```
+----+---------+------+
| id | nombre  | edad |
+----+---------+------+
|  1 | Ana     |   25 | ✅
|  2 | Bob     |   30 | ✅
|  3 | Carlos  |   28 | ❌
+----+---------+------+
```

---

### NOT (Negación)

**Invierte la condición.**

```sql
-- Usuarios que NO tienen 25 años
SELECT * FROM usuarios
WHERE NOT edad = 25;

-- Equivalente:
SELECT * FROM usuarios WHERE edad != 25;
```

---

### Combinación de AND y OR

**Usar paréntesis para claridad.**

```sql
-- Usuarios con edad 25-30 Y activos
SELECT * FROM usuarios
WHERE (edad >= 25 AND edad <= 30) AND activo = 1;

-- Usuarios con edad 25 O nombre 'Ana'
SELECT * FROM usuarios
WHERE edad = 25 OR nombre = 'Ana';

-- Complejo: (edad 25-30 Y activos) O (edad > 50)
SELECT * FROM usuarios
WHERE (edad >= 25 AND edad <= 30 AND activo = 1)
   OR edad > 50;
```

---

## BETWEEN - Rango de valores

Simplifica condiciones con rangos.

```sql
-- Usuarios con edad entre 25 y 30 (inclusive)
SELECT * FROM usuarios
WHERE edad BETWEEN 25 AND 30;

-- Equivalente a:
SELECT * FROM usuarios
WHERE edad >= 25 AND edad <= 30;
```

**Funciona con:**
- Números
- Fechas
- Texto (orden alfabético)

**Ejemplos:**
```sql
-- Precios entre 100 y 500
SELECT * FROM productos WHERE precio BETWEEN 100 AND 500;

-- Fechas entre enero y marzo 2024
SELECT * FROM pedidos
WHERE fecha BETWEEN '2024-01-01' AND '2024-03-31';

-- Nombres entre 'A' y 'M' (alfabéticamente)
SELECT * FROM usuarios
WHERE nombre BETWEEN 'A' AND 'M';
```

---

## IN - Lista de valores

Verificar si un valor está en una lista.

```sql
-- Usuarios con edad 25, 28 o 30
SELECT * FROM usuarios
WHERE edad IN (25, 28, 30);

-- Equivalente a:
SELECT * FROM usuarios
WHERE edad = 25 OR edad = 28 OR edad = 30;
```

**Ventajas de IN:**
- Más legible
- Menos código
- Funciona con subconsultas

**Ejemplos:**
```sql
-- Productos de categorías específicas
SELECT * FROM productos
WHERE categoria IN ('Electrónica', 'Accesorios', 'Gaming');

-- Estados válidos
SELECT * FROM pedidos
WHERE estado IN ('pendiente', 'procesando', 'enviado');
```

---

## NOT IN

**Valores que NO están en la lista.**

```sql
-- Usuarios con edad diferente de 25, 28, 30
SELECT * FROM usuarios
WHERE edad NOT IN (25, 28, 30);
```

---

## LIKE - Patrones de texto

Coincidencia de patrones en strings.

### Comodines

```
%    -- Cualquier secuencia de caracteres (0 o más)
_    -- Un solo carácter
```

---

### Ejemplos con %

```sql
-- Nombres que empiezan con 'A'
SELECT * FROM usuarios WHERE nombre LIKE 'A%';
-- Ana, Alberto, Amanda

-- Nombres que terminan con 'o'
SELECT * FROM usuarios WHERE nombre LIKE '%o';
-- Carlos, Mario, Pedro

-- Nombres que contienen 'ar'
SELECT * FROM usuarios WHERE nombre LIKE '%ar%';
-- Carlos, Mario, Marta

-- Emails de Gmail
SELECT * FROM usuarios WHERE email LIKE '%@gmail.com';
```

---

### Ejemplos con _

```sql
-- Nombres de 3 letras
SELECT * FROM usuarios WHERE nombre LIKE '___';
-- Ana, Bob

-- Nombres que empiezan con 'A' y tienen 4 letras
SELECT * FROM usuarios WHERE nombre LIKE 'A___';
-- Alba, Aldo

-- Nombres con 'n' en segunda posición
SELECT * FROM usuarios WHERE nombre LIKE '_n%';
-- Ana, Enzo
```

---

### Combinación de % y _

```sql
-- Nombres con 'a' en segunda posición y terminan con 'o'
SELECT * FROM usuarios WHERE nombre LIKE '_a%o';
-- Carlos, Pablo, Mario
```

---

### Case-insensitive en SQLite

```sql
-- SQLite es case-insensitive por defecto para LIKE
SELECT * FROM usuarios WHERE nombre LIKE 'ana';
-- Encuentra: Ana, ANA, ana, AnA
```

---

### NOT LIKE

```sql
-- Usuarios sin email de Gmail
SELECT * FROM usuarios WHERE email NOT LIKE '%@gmail.com';
```

---

## IS NULL / IS NOT NULL

Verificar valores nulos.

### IS NULL

```sql
-- Usuarios sin email
SELECT * FROM usuarios WHERE email IS NULL;
```

### IS NOT NULL

```sql
-- Usuarios con email
SELECT * FROM usuarios WHERE email IS NOT NULL;
```

**⚠️ Nunca uses = NULL:**

```sql
-- ❌ MAL (siempre falso)
SELECT * FROM usuarios WHERE email = NULL;

-- ✅ BIEN
SELECT * FROM usuarios WHERE email IS NULL;
```

**Comportamiento de NULL:**
```sql
NULL = NULL   → NULL (no true, no false)
NULL != NULL  → NULL
NULL > 5      → NULL
NULL + 10     → NULL
```

---

## ORDER BY - Ordenar resultados

Ordenar filas por una o más columnas.

### Sintaxis

```sql
SELECT columnas
FROM tabla
ORDER BY columna [ASC|DESC];
```

- `ASC` - Ascendente (menor a mayor) - **default**
- `DESC` - Descendente (mayor a menor)

---

### Ordenar ascendente (ASC)

```sql
-- Usuarios ordenados por edad (menor a mayor)
SELECT * FROM usuarios ORDER BY edad;

-- Equivalente:
SELECT * FROM usuarios ORDER BY edad ASC;
```

**Resultado:**
```
+----+---------+------+
| id | nombre  | edad |
+----+---------+------+
|  4 | Diana   |   20 |
|  1 | Ana     |   25 |
|  3 | Carlos  |   28 |
|  2 | Bob     |   30 |
+----+---------+------+
```

---

### Ordenar descendente (DESC)

```sql
-- Usuarios ordenados por edad (mayor a menor)
SELECT * FROM usuarios ORDER BY edad DESC;
```

**Resultado:**
```
+----+---------+------+
| id | nombre  | edad |
+----+---------+------+
|  2 | Bob     |   30 |
|  3 | Carlos  |   28 |
|  1 | Ana     |   25 |
|  4 | Diana   |   20 |
+----+---------+------+
```

---

### Ordenar por múltiples columnas

```sql
-- Ordenar por categoria (ASC) y luego por precio (DESC)
SELECT * FROM productos
ORDER BY categoria ASC, precio DESC;
```

**Resultado:**
```
+----+------------+-------+--------------+
| id | nombre     | precio| categoria    |
+----+------------+-------+--------------+
|  3 | Laptop Pro | 2000  | Electrónica  | ← categoría A, precio mayor
|  1 | Laptop     | 1000  | Electrónica  | ← categoría A, precio menor
|  4 | Mouse      |   50  | Periféricos  | ← categoría P, precio mayor
|  2 | Teclado    |   30  | Periféricos  | ← categoría P, precio menor
+----+------------+-------+--------------+
```

---

### Ordenar por expresiones

```sql
-- Ordenar por precio con IVA
SELECT nombre, precio, precio * 1.19 AS precio_iva
FROM productos
ORDER BY precio * 1.19 DESC;

-- Ordenar por longitud de nombre
SELECT nombre FROM usuarios
ORDER BY LENGTH(nombre) DESC;
```

---

### Ordenar por posición de columna

```sql
-- Ordenar por la segunda columna (nombre)
SELECT id, nombre, edad FROM usuarios
ORDER BY 2;  -- ← posición 2 = nombre

-- Equivalente:
SELECT id, nombre, edad FROM usuarios
ORDER BY nombre;
```

**⚠️ No recomendado:** Menos legible.

---

## LIMIT - Limitar resultados

Limitar el número de filas devueltas.

### Sintaxis

```sql
SELECT columnas
FROM tabla
LIMIT numero;
```

---

### Ejemplos

```sql
-- Primeros 5 usuarios
SELECT * FROM usuarios LIMIT 5;

-- Top 3 productos más caros
SELECT * FROM productos
ORDER BY precio DESC
LIMIT 3;

-- Usuario más joven
SELECT * FROM usuarios
ORDER BY edad ASC
LIMIT 1;
```

---

### LIMIT con OFFSET

**OFFSET:** Saltar filas.

```sql
-- Sintaxis
SELECT * FROM tabla
LIMIT cantidad OFFSET saltar;
```

**Ejemplos:**
```sql
-- Primeros 10 usuarios
SELECT * FROM usuarios LIMIT 10;

-- Usuarios 11-20 (saltar 10, tomar 10)
SELECT * FROM usuarios LIMIT 10 OFFSET 10;

-- Usuarios 21-30
SELECT * FROM usuarios LIMIT 10 OFFSET 20;
```

**Paginación:**
```sql
-- Página 1 (primeros 10)
SELECT * FROM usuarios LIMIT 10 OFFSET 0;

-- Página 2 (usuarios 11-20)
SELECT * FROM usuarios LIMIT 10 OFFSET 10;

-- Página 3 (usuarios 21-30)
SELECT * FROM usuarios LIMIT 10 OFFSET 20;

-- Fórmula:
-- OFFSET = (pagina - 1) * items_por_pagina
```

---

## Combinaciones prácticas

### Filtrar, ordenar y limitar

```sql
-- Top 5 usuarios activos mayores de 25, ordenados por edad
SELECT nombre, edad
FROM usuarios
WHERE edad > 25 AND activo = 1
ORDER BY edad DESC
LIMIT 5;
```

---

### Búsqueda con patrones

```sql
-- Usuarios con Gmail ordenados alfabéticamente
SELECT nombre, email
FROM usuarios
WHERE email LIKE '%@gmail.com'
ORDER BY nombre ASC;
```

---

### Productos en rango de precio

```sql
-- Productos entre $100 y $500, más baratos primero
SELECT nombre, precio
FROM productos
WHERE precio BETWEEN 100 AND 500
ORDER BY precio ASC
LIMIT 10;
```

---

## Caso práctico completo

### Base de datos de productos

```sql
CREATE TABLE productos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    categoria TEXT,
    precio REAL CHECK (precio > 0),
    stock INTEGER CHECK (stock >= 0),
    activo INTEGER DEFAULT 1
);

INSERT INTO productos (nombre, categoria, precio, stock, activo)
VALUES 
    ('Laptop Dell', 'Electrónica', 1200, 10, 1),
    ('Mouse Logitech', 'Accesorios', 25, 50, 1),
    ('Teclado Mecánico', 'Accesorios', 80, 30, 1),
    ('Monitor LG 24"', 'Electrónica', 200, 15, 1),
    ('Webcam HD', 'Accesorios', 50, 0, 1),
    ('Laptop HP', 'Electrónica', 900, 5, 0),
    ('Mouse Inalámbrico', 'Accesorios', 15, 100, 1),
    ('Monitor Samsung 27"', 'Electrónica', 300, 8, 1);
```

---

### Consultas prácticas

**1. Productos activos con stock:**
```sql
SELECT nombre, precio, stock
FROM productos
WHERE activo = 1 AND stock > 0;
```

**2. Top 3 productos más caros:**
```sql
SELECT nombre, precio
FROM productos
ORDER BY precio DESC
LIMIT 3;
```

**3. Productos de Electrónica entre $200 y $1000:**
```sql
SELECT nombre, precio
FROM productos
WHERE categoria = 'Electrónica'
  AND precio BETWEEN 200 AND 1000
ORDER BY precio ASC;
```

**4. Productos con poco stock (< 10):**
```sql
SELECT nombre, stock
FROM productos
WHERE stock < 10 AND stock > 0
ORDER BY stock ASC;
```

**5. Buscar productos con "Mouse" en el nombre:**
```sql
SELECT nombre, precio
FROM productos
WHERE nombre LIKE '%Mouse%';
```

**6. Productos sin stock:**
```sql
SELECT nombre, categoria
FROM productos
WHERE stock = 0;
```

**7. Paginación (página 2, 3 items por página):**
```sql
SELECT nombre, precio
FROM productos
ORDER BY nombre
LIMIT 3 OFFSET 3;
```

---

## Funciones útiles en WHERE

### LENGTH() - Longitud de texto

```sql
-- Nombres con más de 10 caracteres
SELECT nombre FROM usuarios
WHERE LENGTH(nombre) > 10;
```

### UPPER() / LOWER() - Cambiar mayúsculas/minúsculas

```sql
-- Buscar case-sensitive
SELECT * FROM usuarios
WHERE UPPER(email) = 'ANA@EXAMPLE.COM';
```

### TRIM() - Quitar espacios

```sql
-- Comparar sin espacios
SELECT * FROM usuarios
WHERE TRIM(nombre) = 'Ana';
```

### SUBSTR() - Subcadena

```sql
-- Usuarios cuyo nombre empieza con 'An'
SELECT * FROM usuarios
WHERE SUBSTR(nombre, 1, 2) = 'An';
```

---

## Buenas prácticas

### ✅ Usar BETWEEN para rangos

```sql
-- BIEN
SELECT * FROM usuarios WHERE edad BETWEEN 25 AND 30;

-- Evitar
SELECT * FROM usuarios WHERE edad >= 25 AND edad <= 30;
```

### ✅ Usar IN para múltiples valores

```sql
-- BIEN
SELECT * FROM usuarios WHERE edad IN (25, 28, 30);

-- Evitar
SELECT * FROM usuarios WHERE edad = 25 OR edad = 28 OR edad = 30;
```

### ✅ Siempre ORDER BY con LIMIT

```sql
-- BIEN (orden predecible)
SELECT * FROM productos ORDER BY precio DESC LIMIT 10;

-- MAL (orden aleatorio)
SELECT * FROM productos LIMIT 10;
```

### ✅ Paréntesis en condiciones complejas

```sql
-- BIEN (claro)
WHERE (edad > 25 AND activo = 1) OR (edad < 18)

-- Evitar (ambiguo)
WHERE edad > 25 AND activo = 1 OR edad < 18
```

---

## Errores comunes

### ❌ Usar = NULL

```sql
-- MAL
SELECT * FROM usuarios WHERE email = NULL;

-- BIEN
SELECT * FROM usuarios WHERE email IS NULL;
```

### ❌ LIKE sin %

```sql
-- Busca exactamente 'ana' (como =)
SELECT * FROM usuarios WHERE nombre LIKE 'ana';

-- Para buscar que contenga 'ana'
SELECT * FROM usuarios WHERE nombre LIKE '%ana%';
```

### ❌ Olvidar ORDER BY con LIMIT

```sql
-- Resultados impredecibles
SELECT * FROM productos LIMIT 10;

-- Resultados consistentes
SELECT * FROM productos ORDER BY id LIMIT 10;
```

---

## Resumen

**WHERE:**
- Filtra filas basándose en condiciones
- Operadores: `=`, `!=`, `>`, `<`, `>=`, `<=`
- Lógicos: `AND`, `OR`, `NOT`
- Especiales: `BETWEEN`, `IN`, `LIKE`, `IS NULL`

**ORDER BY:**
- Ordena resultados
- `ASC` - ascendente (default)
- `DESC` - descendente
- Múltiples columnas separadas por coma

**LIMIT:**
- Limita cantidad de filas
- `LIMIT n` - primeras n filas
- `LIMIT n OFFSET m` - saltar m, tomar n
- Útil para paginación

**Orden de ejecución:**
```sql
FROM tabla           -- 1. Tabla
WHERE condicion      -- 2. Filtrar
ORDER BY columna     -- 3. Ordenar
LIMIT numero         -- 4. Limitar
```

**Próximo módulo:** JOINs - Relacionar tablas

---

## Ejercicios

### Ejercicio 1

Usuarios con edad entre 20 y 30, ordenados por nombre.

**Solución:**
```sql
SELECT * FROM usuarios
WHERE edad BETWEEN 20 AND 30
ORDER BY nombre;
```

---

### Ejercicio 2

Top 5 productos más baratos con stock.

**Solución:**
```sql
SELECT nombre, precio, stock
FROM productos
WHERE stock > 0
ORDER BY precio ASC
LIMIT 5;
```

---

### Ejercicio 3

Productos de categoría 'Electrónica' o 'Accesorios' con precio > 50.

**Solución:**
```sql
SELECT * FROM productos
WHERE categoria IN ('Electrónica', 'Accesorios')
  AND precio > 50;
```

---

### Ejercicio 4

Usuarios con email de Gmail o sin email.

**Solución:**
```sql
SELECT * FROM usuarios
WHERE email LIKE '%@gmail.com'
   OR email IS NULL;
```

---

### Ejercicio 5

Paginación: página 3, 10 items por página.

**Solución:**
```sql
SELECT * FROM productos
ORDER BY id
LIMIT 10 OFFSET 20;  -- (3-1) * 10 = 20
```

---

## Recursos

- **[WHERE Clause](https://www.sqlitetutorial.net/sqlite-where/)** - Tutorial completo
- **[ORDER BY](https://www.sqlitetutorial.net/sqlite-order-by/)** - Ordenamiento
- **[LIMIT](https://www.sqlitetutorial.net/sqlite-limit/)** - Limitación
- **[LIKE](https://www.sqlitetutorial.net/sqlite-like/)** - Patrones

Ahora dominas el **filtrado y ordenamiento** en SQL. 🎯

# GROUP BY y HAVING

> **Agregar datos por grupos y filtrar agregaciones**

---

## GROUP BY - Agrupar resultados

**GROUP BY** agrupa filas que tienen valores iguales en columnas especificadas.

**Analogía:**  
Como organizar productos por categoría y contar cuántos hay en cada una.

---

## Sintaxis básica

```sql
SELECT columna_agrupacion, FUNCION_AGREGACION(columna)
FROM tabla
GROUP BY columna_agrupacion;
```

---

## Ejemplo simple

### Sin GROUP BY

```sql
-- ¿Cuántos productos hay en total?
SELECT COUNT(*) FROM productos;
-- Resultado: 5
```

### Con GROUP BY

```sql
-- ¿Cuántos productos hay por categoría?
SELECT categoria, COUNT(*) AS total
FROM productos
GROUP BY categoria;
```

**Resultado:**
```
+--------------+-------+
| categoria    | total |
+--------------+-------+
| Electrónica  |     3 |
| Accesorios   |     2 |
+--------------+-------+
```

---

## Caso práctico: Tienda

### Datos

```sql
CREATE TABLE productos (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    categoria TEXT,
    precio REAL,
    stock INTEGER
);

INSERT INTO productos (id, nombre, categoria, precio, stock) VALUES
    (1, 'Laptop Dell', 'Electrónica', 1200, 10),
    (2, 'Mouse Logitech', 'Accesorios', 25, 50),
    (3, 'Teclado Mecánico', 'Accesorios', 80, 30),
    (4, 'Monitor LG', 'Electrónica', 200, 15),
    (5, 'Webcam HD', 'Electrónica', 50, 20);
```

---

### Queries con GROUP BY

**1. Productos por categoría:**

```sql
SELECT categoria, COUNT(*) AS total_productos
FROM productos
GROUP BY categoria;
```

**Resultado:**
```
+--------------+-----------------+
| categoria    | total_productos |
+--------------+-----------------+
| Electrónica  |               3 |
| Accesorios   |               2 |
+--------------+-----------------+
```

---

**2. Valor promedio por categoría:**

```sql
SELECT 
    categoria,
    COUNT(*) AS productos,
    AVG(precio) AS precio_promedio,
    MIN(precio) AS mas_barato,
    MAX(precio) AS mas_caro
FROM productos
GROUP BY categoria;
```

**Resultado:**
```
+--------------+-----------+------------------+------------+-----------+
| categoria    | productos | precio_promedio  | mas_barato | mas_caro  |
+--------------+-----------+------------------+------------+-----------+
| Electrónica  |         3 |           483.33 |         50 |      1200 |
| Accesorios   |         2 |            52.50 |         25 |        80 |
+--------------+-----------+------------------+------------+-----------+
```

---

**3. Stock total por categoría:**

```sql
SELECT 
    categoria,
    SUM(stock) AS stock_total,
    AVG(stock) AS stock_promedio
FROM productos
GROUP BY categoria;
```

**Resultado:**
```
+--------------+-------------+-----------------+
| categoria    | stock_total | stock_promedio  |
+--------------+-------------+-----------------+
| Electrónica  |          45 |            15.0 |
| Accesorios   |          80 |            40.0 |
+--------------+-------------+-----------------+
```

---

## GROUP BY con múltiples columnas

```sql
CREATE TABLE ventas (
    id INTEGER PRIMARY KEY,
    categoria TEXT,
    producto TEXT,
    cantidad INTEGER,
    mes TEXT
);

INSERT INTO ventas VALUES
    (1, 'Electrónica', 'Laptop', 5, 'Enero'),
    (2, 'Electrónica', 'Laptop', 3, 'Febrero'),
    (3, 'Electrónica', 'Monitor', 10, 'Enero'),
    (4, 'Accesorios', 'Mouse', 50, 'Enero'),
    (5, 'Accesorios', 'Mouse', 30, 'Febrero');

-- Ventas por categoría y mes
SELECT 
    categoria,
    mes,
    SUM(cantidad) AS total_vendido
FROM ventas
GROUP BY categoria, mes
ORDER BY categoria, mes;
```

**Resultado:**
```
+--------------+---------+---------------+
| categoria    | mes     | total_vendido |
+--------------+---------+---------------+
| Accesorios   | Enero   |            50 |
| Accesorios   | Febrero |            30 |
| Electrónica  | Enero   |            15 | (5 laptops + 10 monitores)
| Electrónica  | Febrero |             3 |
+--------------+---------+---------------+
```

---

## GROUP BY con JOINs

### Escenario: Pedidos

```sql
CREATE TABLE clientes (
    id INTEGER PRIMARY KEY,
    nombre TEXT
);

CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente_id INTEGER,
    total REAL,
    fecha DATE,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

INSERT INTO clientes VALUES (1, 'Ana'), (2, 'Bob'), (3, 'Carlos');

INSERT INTO pedidos (id, cliente_id, total, fecha) VALUES
    (1, 1, 1200, '2024-01-15'),
    (2, 1, 50, '2024-01-20'),
    (3, 2, 300, '2024-01-18'),
    (4, 2, 150, '2024-01-22'),
    (5, 3, 800, '2024-01-25');
```

---

### Query: Pedidos por cliente

```sql
SELECT 
    c.nombre,
    COUNT(p.id) AS num_pedidos,
    SUM(p.total) AS total_gastado,
    AVG(p.total) AS ticket_promedio
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre
ORDER BY total_gastado DESC;
```

**Resultado:**
```
+---------+-------------+---------------+-----------------+
| nombre  | num_pedidos | total_gastado | ticket_promedio |
+---------+-------------+---------------+-----------------+
| Ana     |           2 |          1250 |             625 |
| Carlos  |           1 |           800 |             800 |
| Bob     |           2 |           450 |             225 |
+---------+-------------+---------------+-----------------+
```

---

## HAVING - Filtrar grupos

**HAVING** filtra grupos **después** de aplicar GROUP BY.

**Diferencia con WHERE:**
- `WHERE` filtra filas **antes** de agrupar
- `HAVING` filtra grupos **después** de agrupar

---

### Sintaxis

```sql
SELECT columna_agrupacion, FUNCION_AGREGACION(columna)
FROM tabla
WHERE condicion_filas
GROUP BY columna_agrupacion
HAVING condicion_grupos;
```

---

### Ejemplo: WHERE vs HAVING

```sql
-- WHERE: filtrar productos antes de agrupar
SELECT categoria, COUNT(*) AS total
FROM productos
WHERE precio > 50  -- ← filtra productos individuales
GROUP BY categoria;

-- HAVING: filtrar grupos después de agrupar
SELECT categoria, COUNT(*) AS total
FROM productos
GROUP BY categoria
HAVING COUNT(*) > 2;  -- ← filtra grupos (categorías)
```

---

### Ejemplos prácticos con HAVING

**1. Categorías con más de 2 productos:**

```sql
SELECT 
    categoria,
    COUNT(*) AS total_productos
FROM productos
GROUP BY categoria
HAVING COUNT(*) > 2;
```

**Resultado:**
```
+--------------+-----------------+
| categoria    | total_productos |
+--------------+-----------------+
| Electrónica  |               3 |
+--------------+-----------------+
```

---

**2. Clientes con gasto total > $500:**

```sql
SELECT 
    c.nombre,
    SUM(p.total) AS total_gastado
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre
HAVING SUM(p.total) > 500
ORDER BY total_gastado DESC;
```

**Resultado:**
```
+---------+---------------+
| nombre  | total_gastado |
+---------+---------------+
| Ana     |          1250 |
| Carlos  |           800 |
+---------+---------------+
```

---

**3. Categorías con precio promedio > $100:**

```sql
SELECT 
    categoria,
    AVG(precio) AS precio_promedio,
    COUNT(*) AS productos
FROM productos
GROUP BY categoria
HAVING AVG(precio) > 100;
```

**Resultado:**
```
+--------------+------------------+-----------+
| categoria    | precio_promedio  | productos |
+--------------+------------------+-----------+
| Electrónica  |           483.33 |         3 |
+--------------+------------------+-----------+
```

---

**4. Productos con stock promedio > 20:**

```sql
SELECT 
    categoria,
    AVG(stock) AS stock_promedio
FROM productos
GROUP BY categoria
HAVING AVG(stock) > 20;
```

---

## WHERE y HAVING juntos

```sql
-- Clientes activos con más de 1 pedido en enero
SELECT 
    c.nombre,
    COUNT(p.id) AS pedidos_enero
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id
WHERE p.fecha BETWEEN '2024-01-01' AND '2024-01-31'  -- ← filtra filas
GROUP BY c.id, c.nombre
HAVING COUNT(p.id) > 1;  -- ← filtra grupos
```

**Lógica:**
1. `WHERE`: Filtra solo pedidos de enero
2. `GROUP BY`: Agrupa por cliente
3. `HAVING`: Filtra solo clientes con más de 1 pedido

---

## Orden de ejecución SQL

**Orden que escribe:**
```sql
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

**Orden que ejecuta la BD:**
```
1. FROM      - Identifica tabla(s)
2. WHERE     - Filtra filas
3. GROUP BY  - Agrupa
4. HAVING    - Filtra grupos
5. SELECT    - Selecciona columnas
6. ORDER BY  - Ordena
7. LIMIT     - Limita resultados
```

---

## Ejemplos completos

### E-commerce: Análisis de ventas

```sql
CREATE TABLE categorias (
    id INTEGER PRIMARY KEY,
    nombre TEXT
);

CREATE TABLE productos (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    categoria_id INTEGER,
    precio REAL,
    FOREIGN KEY (categoria_id) REFERENCES categorias(id)
);

CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente_id INTEGER,
    fecha DATE
);

CREATE TABLE items_pedido (
    id INTEGER PRIMARY KEY,
    pedido_id INTEGER,
    producto_id INTEGER,
    cantidad INTEGER,
    FOREIGN KEY (pedido_id) REFERENCES pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES productos(id)
);

-- Datos de ejemplo
INSERT INTO categorias VALUES (1, 'Electrónica'), (2, 'Ropa'), (3, 'Hogar');

INSERT INTO productos VALUES
    (1, 'Laptop', 1, 1000),
    (2, 'Mouse', 1, 20),
    (3, 'Teclado', 1, 50),
    (4, 'Camisa', 2, 30),
    (5, 'Pantalón', 2, 40),
    (6, 'Mesa', 3, 200);

INSERT INTO pedidos VALUES
    (1, 1, '2024-01-15'),
    (2, 2, '2024-01-16'),
    (3, 1, '2024-01-17');

INSERT INTO items_pedido VALUES
    (1, 1, 1, 1),  -- Pedido 1: 1 laptop
    (2, 1, 2, 2),  -- Pedido 1: 2 mouse
    (3, 2, 4, 3),  -- Pedido 2: 3 camisas
    (4, 3, 3, 1),  -- Pedido 3: 1 teclado
    (5, 3, 5, 2);  -- Pedido 3: 2 pantalones
```

---

### Queries analíticas

**1. Ventas por categoría:**

```sql
SELECT 
    cat.nombre AS categoria,
    COUNT(DISTINCT ped.id) AS num_pedidos,
    SUM(i.cantidad) AS unidades_vendidas,
    SUM(i.cantidad * prod.precio) AS ingresos_totales
FROM categorias cat
INNER JOIN productos prod ON cat.id = prod.categoria_id
INNER JOIN items_pedido i ON prod.id = i.producto_id
INNER JOIN pedidos ped ON i.pedido_id = ped.id
GROUP BY cat.id, cat.nombre
ORDER BY ingresos_totales DESC;
```

**Resultado:**
```
+--------------+-------------+--------------------+------------------+
| categoria    | num_pedidos | unidades_vendidas  | ingresos_totales |
+--------------+-------------+--------------------+------------------+
| Electrónica  |           2 |                  4 |             1090 |
| Ropa         |           2 |                  5 |              170 |
| Hogar        |           0 |               NULL |             NULL |
+--------------+-------------+--------------------+------------------+
```

---

**2. Top 3 productos más vendidos:**

```sql
SELECT 
    prod.nombre,
    SUM(i.cantidad) AS total_vendido,
    SUM(i.cantidad * prod.precio) AS ingresos
FROM productos prod
INNER JOIN items_pedido i ON prod.id = i.producto_id
GROUP BY prod.id, prod.nombre
ORDER BY total_vendido DESC
LIMIT 3;
```

---

**3. Categorías con más de $100 en ventas:**

```sql
SELECT 
    cat.nombre,
    SUM(i.cantidad * prod.precio) AS ingresos
FROM categorias cat
INNER JOIN productos prod ON cat.id = prod.categoria_id
INNER JOIN items_pedido i ON prod.id = i.producto_id
GROUP BY cat.id, cat.nombre
HAVING SUM(i.cantidad * prod.precio) > 100
ORDER BY ingresos DESC;
```

---

## Funciones avanzadas con GROUP BY

### COUNT con DISTINCT

```sql
-- Número de clientes únicos por categoría
SELECT 
    cat.nombre,
    COUNT(DISTINCT ped.cliente_id) AS clientes_unicos
FROM categorias cat
INNER JOIN productos prod ON cat.id = prod.categoria_id
INNER JOIN items_pedido i ON prod.id = i.producto_id
INNER JOIN pedidos ped ON i.pedido_id = ped.id
GROUP BY cat.id, cat.nombre;
```

---

### Expresiones en GROUP BY

```sql
-- Ventas por rango de precio
SELECT 
    CASE
        WHEN precio < 50 THEN 'Barato'
        WHEN precio BETWEEN 50 AND 200 THEN 'Medio'
        ELSE 'Caro'
    END AS rango_precio,
    COUNT(*) AS num_productos,
    AVG(precio) AS precio_promedio
FROM productos
GROUP BY rango_precio;
```

---

### GROUP BY con fechas

```sql
-- Pedidos por mes
SELECT 
    strftime('%Y-%m', fecha) AS mes,
    COUNT(*) AS num_pedidos
FROM pedidos
GROUP BY mes
ORDER BY mes;
```

---

## Buenas prácticas

### ✅ Incluir columna agrupada en SELECT

```sql
-- BIEN
SELECT categoria, COUNT(*)
FROM productos
GROUP BY categoria;

-- ❌ ERROR en la mayoría de BD (SQLite permite pero no recomendado)
SELECT nombre, COUNT(*)
FROM productos
GROUP BY categoria;
```

### ✅ Usar HAVING para filtrar agregaciones

```sql
-- BIEN
SELECT categoria, COUNT(*) AS total
FROM productos
GROUP BY categoria
HAVING COUNT(*) > 2;

-- ❌ ERROR (no puedes usar agregaciones en WHERE)
SELECT categoria, COUNT(*) AS total
FROM productos
WHERE COUNT(*) > 2
GROUP BY categoria;
```

### ✅ WHERE antes de GROUP BY para performance

```sql
-- BIEN (filtra antes de agrupar)
SELECT categoria, COUNT(*)
FROM productos
WHERE activo = 1
GROUP BY categoria;

-- Menos eficiente (agrupa todo, luego filtra)
SELECT categoria, COUNT(*)
FROM productos
GROUP BY categoria
HAVING activo = 1;  -- ❌ esto no funcionaría
```

---

## Errores comunes

### ❌ Columna no agregada sin GROUP BY

```sql
-- ❌ ERROR
SELECT nombre, COUNT(*) FROM usuarios;
-- ¿Qué nombre mostrar?

-- ✅ BIEN
SELECT nombre, COUNT(*) FROM usuarios GROUP BY nombre;
```

### ❌ Usar alias en HAVING

```sql
-- ❌ Puede no funcionar
SELECT categoria, COUNT(*) AS total
FROM productos
GROUP BY categoria
HAVING total > 2;

-- ✅ BIEN
SELECT categoria, COUNT(*) AS total
FROM productos
GROUP BY categoria
HAVING COUNT(*) > 2;
```

### ❌ WHERE con agregaciones

```sql
-- ❌ ERROR
SELECT categoria, AVG(precio)
FROM productos
WHERE AVG(precio) > 100
GROUP BY categoria;

-- ✅ BIEN
SELECT categoria, AVG(precio)
FROM productos
GROUP BY categoria
HAVING AVG(precio) > 100;
```

---

## Resumen

**GROUP BY:**
- Agrupa filas con valores iguales
- Se usa con funciones de agregación
- Sintaxis: `GROUP BY columna1, columna2, ...`

**HAVING:**
- Filtra grupos después de GROUP BY
- Se usa con funciones de agregación
- Sintaxis: `HAVING condicion`

**Diferencia WHERE vs HAVING:**
- `WHERE` - Filtra filas individuales (antes de agrupar)
- `HAVING` - Filtra grupos (después de agrupar)

**Orden de ejecución:**
```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

**Próximo módulo:** Subconsultas (queries dentro de queries)

---

## Ejercicios

### Ejercicio 1

Productos por categoría con stock total.

**Solución:**
```sql
SELECT 
    categoria,
    COUNT(*) AS productos,
    SUM(stock) AS stock_total
FROM productos
GROUP BY categoria;
```

---

### Ejercicio 2

Categorías con más de 2 productos.

**Solución:**
```sql
SELECT categoria, COUNT(*) AS productos
FROM productos
GROUP BY categoria
HAVING COUNT(*) > 2;
```

---

### Ejercicio 3

Clientes con gasto total superior al promedio.

**Solución:**
```sql
SELECT 
    c.nombre,
    SUM(p.total) AS total_gastado
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre
HAVING SUM(p.total) > (SELECT AVG(total_cliente) FROM (
    SELECT SUM(total) AS total_cliente
    FROM pedidos
    GROUP BY cliente_id
));
```

---

### Ejercicio 4

Top 5 categorías con mayores ventas.

**Solución:**
```sql
SELECT 
    cat.nombre,
    SUM(i.cantidad * prod.precio) AS ventas_totales
FROM categorias cat
INNER JOIN productos prod ON cat.id = prod.categoria_id
INNER JOIN items_pedido i ON prod.id = i.producto_id
GROUP BY cat.id, cat.nombre
ORDER BY ventas_totales DESC
LIMIT 5;
```

---

## Recursos

- **[GROUP BY](https://www.sqlitetutorial.net/sqlite-group-by/)** - Tutorial completo
- **[HAVING](https://www.sqlitetutorial.net/sqlite-having/)** - Filtrar grupos
- **[WHERE vs HAVING](https://www.w3schools.com/sql/sql_having.asp)** - Diferencias

Ahora dominas **GROUP BY y HAVING**. 📊

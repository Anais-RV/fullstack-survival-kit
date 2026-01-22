# Subconsultas (Subqueries)

> **Queries dentro de queries para consultas complejas**

---

## ¿Qué es una subconsulta?

Una **subconsulta** es una consulta SQL anidada dentro de otra consulta.

**Analogía:**  
Como una función dentro de otra función en programación:
```python
resultado = funcion_externa(funcion_interna())
```

---

## Tipos de subconsultas

1. **En WHERE** - Filtrar basándose en otra query
2. **En FROM** - Usar resultado como tabla temporal
3. **En SELECT** - Agregar columna calculada
4. **Correlacionadas** - Referencian la query externa

---

## Subconsultas en WHERE

Filtrar filas basándose en el resultado de otra consulta.

### Sintaxis

```sql
SELECT columnas
FROM tabla
WHERE columna OPERADOR (
    SELECT columna
    FROM otra_tabla
    WHERE condicion
);
```

---

### Ejemplo: Productos más caros que el promedio

```sql
-- Datos
CREATE TABLE productos (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    precio REAL
);

INSERT INTO productos VALUES
    (1, 'Laptop', 1200),
    (2, 'Mouse', 25),
    (3, 'Teclado', 80),
    (4, 'Monitor', 200),
    (5, 'Webcam', 50);

-- Query
SELECT nombre, precio
FROM productos
WHERE precio > (
    SELECT AVG(precio)
    FROM productos
);
```

**Resultado:**
```
+---------+--------+
| nombre  | precio |
+---------+--------+
| Laptop  |   1200 |
| Monitor |    200 |
+---------+--------+
```

**Explicación:**
1. Subconsulta: `AVG(precio) = 311`
2. Query externa: `WHERE precio > 311`

---

### Ejemplo: Clientes sin pedidos

```sql
-- Alternativa a LEFT JOIN + WHERE NULL
SELECT nombre
FROM clientes
WHERE id NOT IN (
    SELECT DISTINCT cliente_id
    FROM pedidos
);
```

---

## Subconsultas con operadores

### IN y NOT IN

```sql
-- Productos de categorías con más de 2 productos
SELECT nombre, precio
FROM productos
WHERE categoria_id IN (
    SELECT categoria_id
    FROM productos
    GROUP BY categoria_id
    HAVING COUNT(*) > 2
);
```

### Comparaciones (=, >, <, >=, <=, !=)

```sql
-- Empleado con mayor salario
SELECT nombre, salario
FROM empleados
WHERE salario = (
    SELECT MAX(salario)
    FROM empleados
);
```

---

## Subconsultas en FROM (Derived Tables)

Usar el resultado de una query como tabla temporal.

### Sintaxis

```sql
SELECT columnas
FROM (
    SELECT columnas
    FROM tabla
    WHERE condicion
) AS alias
WHERE condicion;
```

---

### Ejemplo: Promedio de promedios

```sql
-- Calcular precio promedio por categoría, luego promedio general
SELECT AVG(precio_promedio) AS promedio_general
FROM (
    SELECT categoria_id, AVG(precio) AS precio_promedio
    FROM productos
    GROUP BY categoria_id
) AS promedios_categoria;
```

---

### Ejemplo: Top 3 productos por categoría

```sql
-- Para cada categoría, seleccionar los 3 productos más caros
SELECT 
    categoria,
    nombre,
    precio,
    ranking
FROM (
    SELECT 
        categoria,
        nombre,
        precio,
        ROW_NUMBER() OVER (PARTITION BY categoria ORDER BY precio DESC) AS ranking
    FROM productos
) AS productos_rankeados
WHERE ranking <= 3;
```

---

## Subconsultas en SELECT

Agregar columna calculada usando otra query.

### Sintaxis

```sql
SELECT 
    columna,
    (SELECT ... FROM ... WHERE ...) AS nueva_columna
FROM tabla;
```

---

### Ejemplo: Total de pedidos por cliente

```sql
SELECT 
    c.nombre,
    c.email,
    (SELECT COUNT(*) 
     FROM pedidos p 
     WHERE p.cliente_id = c.id) AS num_pedidos,
    (SELECT SUM(total) 
     FROM pedidos p 
     WHERE p.cliente_id = c.id) AS total_gastado
FROM clientes c;
```

**Resultado:**
```
+---------+------------------+-------------+---------------+
| nombre  | email            | num_pedidos | total_gastado |
+---------+------------------+-------------+---------------+
| Ana     | ana@email.com    |           2 |          1250 |
| Bob     | bob@email.com    |           2 |           450 |
| Carlos  | carlos@email.com |           1 |           800 |
| Diana   | diana@email.com  |           0 |          NULL |
+---------+------------------+-------------+---------------+
```

---

## Subconsultas correlacionadas

**Subconsulta correlacionada**: Referencia columnas de la query externa.

**Característica:**  
Se ejecuta **una vez por cada fila** de la query externa (puede ser lento).

---

### Sintaxis

```sql
SELECT columnas
FROM tabla AS t1
WHERE columna OPERADOR (
    SELECT columna
    FROM otra_tabla AS t2
    WHERE t2.columna = t1.columna  -- ← Correlación
);
```

---

### Ejemplo: Empleados con salario > promedio de su departamento

```sql
CREATE TABLE empleados (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    departamento TEXT,
    salario REAL
);

INSERT INTO empleados VALUES
    (1, 'Ana', 'Ventas', 3000),
    (2, 'Bob', 'Ventas', 2500),
    (3, 'Carlos', 'IT', 4000),
    (4, 'Diana', 'IT', 3500),
    (5, 'Eva', 'RRHH', 2800);

-- Query
SELECT 
    nombre,
    departamento,
    salario,
    (SELECT AVG(salario) 
     FROM empleados e2 
     WHERE e2.departamento = e1.departamento) AS promedio_dpto
FROM empleados e1
WHERE salario > (
    SELECT AVG(salario)
    FROM empleados e2
    WHERE e2.departamento = e1.departamento
);
```

**Resultado:**
```
+--------+--------------+---------+----------------+
| nombre | departamento | salario | promedio_dpto  |
+--------+--------------+---------+----------------+
| Ana    | Ventas       |    3000 |           2750 |
| Carlos | IT           |    4000 |           3750 |
+--------+--------------+---------+----------------+
```

---

### Ejemplo: Productos más caros por categoría

```sql
SELECT 
    p1.nombre,
    p1.categoria,
    p1.precio
FROM productos p1
WHERE p1.precio = (
    SELECT MAX(p2.precio)
    FROM productos p2
    WHERE p2.categoria = p1.categoria
);
```

---

## EXISTS y NOT EXISTS

**EXISTS**: Verifica si la subconsulta devuelve **al menos una fila**.

**Ventaja**: Más eficiente que IN porque para en la primera coincidencia.

---

### Sintaxis

```sql
SELECT columnas
FROM tabla
WHERE EXISTS (
    SELECT 1
    FROM otra_tabla
    WHERE condicion
);
```

---

### Ejemplo: Clientes con pedidos

```sql
SELECT nombre
FROM clientes c
WHERE EXISTS (
    SELECT 1
    FROM pedidos p
    WHERE p.cliente_id = c.id
);
```

---

### Ejemplo: Clientes sin pedidos

```sql
SELECT nombre
FROM clientes c
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos p
    WHERE p.cliente_id = c.id
);
```

**Equivalente con LEFT JOIN:**
```sql
SELECT c.nombre
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
WHERE p.id IS NULL;
```

---

## ANY y ALL

### ANY (SOME)

Compara con **cualquier** valor de la subconsulta.

```sql
-- Empleados con salario mayor a CUALQUIER empleado de Ventas
SELECT nombre, salario
FROM empleados
WHERE salario > ANY (
    SELECT salario
    FROM empleados
    WHERE departamento = 'Ventas'
);
```

---

### ALL

Compara con **todos** los valores de la subconsulta.

```sql
-- Empleados con salario mayor a TODOS los empleados de Ventas
SELECT nombre, salario
FROM empleados
WHERE salario > ALL (
    SELECT salario
    FROM empleados
    WHERE departamento = 'Ventas'
);
```

---

## Caso práctico: E-commerce

### Escenario completo

```sql
CREATE TABLE clientes (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    ciudad TEXT
);

CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente_id INTEGER,
    total REAL,
    fecha DATE,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

CREATE TABLE productos (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    categoria TEXT,
    precio REAL
);

CREATE TABLE items_pedido (
    id INTEGER PRIMARY KEY,
    pedido_id INTEGER,
    producto_id INTEGER,
    cantidad INTEGER,
    FOREIGN KEY (pedido_id) REFERENCES pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES productos(id)
);

-- Datos
INSERT INTO clientes VALUES 
    (1, 'Ana', 'Madrid'),
    (2, 'Bob', 'Barcelona'),
    (3, 'Carlos', 'Valencia'),
    (4, 'Diana', 'Madrid');

INSERT INTO pedidos VALUES
    (1, 1, 1200, '2024-01-15'),
    (2, 1, 50, '2024-01-20'),
    (3, 2, 300, '2024-01-18'),
    (4, 2, 150, '2024-01-22'),
    (5, 3, 800, '2024-01-25');

INSERT INTO productos VALUES
    (1, 'Laptop', 'Electrónica', 1000),
    (2, 'Mouse', 'Accesorios', 20),
    (3, 'Teclado', 'Accesorios', 50),
    (4, 'Monitor', 'Electrónica', 200),
    (5, 'Webcam', 'Electrónica', 80);

INSERT INTO items_pedido VALUES
    (1, 1, 1, 1),  -- Ana: 1 laptop
    (2, 1, 2, 2),  -- Ana: 2 mouse
    (3, 2, 3, 1),  -- Ana: 1 teclado
    (4, 3, 4, 1),  -- Bob: 1 monitor
    (5, 4, 2, 5),  -- Bob: 5 mouse
    (6, 5, 5, 10); -- Carlos: 10 webcam
```

---

### Query 1: Clientes con gasto > promedio

```sql
SELECT 
    nombre,
    (SELECT SUM(total) FROM pedidos WHERE cliente_id = c.id) AS total_gastado
FROM clientes c
WHERE (SELECT SUM(total) FROM pedidos WHERE cliente_id = c.id) > (
    SELECT AVG(total_cliente) 
    FROM (
        SELECT SUM(total) AS total_cliente
        FROM pedidos
        GROUP BY cliente_id
    )
)
ORDER BY total_gastado DESC;
```

---

### Query 2: Productos nunca comprados

```sql
SELECT nombre, precio
FROM productos
WHERE id NOT IN (
    SELECT DISTINCT producto_id
    FROM items_pedido
);
```

---

### Query 3: Top cliente por ciudad

```sql
-- Cliente con mayor gasto en cada ciudad
SELECT 
    c.ciudad,
    c.nombre,
    (SELECT SUM(total) FROM pedidos WHERE cliente_id = c.id) AS total_gastado
FROM clientes c
WHERE (SELECT SUM(total) FROM pedidos WHERE cliente_id = c.id) = (
    SELECT MAX(gasto_cliente)
    FROM (
        SELECT SUM(p.total) AS gasto_cliente
        FROM clientes c2
        INNER JOIN pedidos p ON c2.id = p.cliente_id
        WHERE c2.ciudad = c.ciudad
        GROUP BY c2.id
    )
);
```

---

### Query 4: Productos más vendidos que el promedio

```sql
-- Productos con ventas superiores al promedio
SELECT 
    prod.nombre,
    SUM(i.cantidad) AS total_vendido
FROM productos prod
INNER JOIN items_pedido i ON prod.id = i.producto_id
GROUP BY prod.id, prod.nombre
HAVING SUM(i.cantidad) > (
    SELECT AVG(ventas_producto)
    FROM (
        SELECT SUM(cantidad) AS ventas_producto
        FROM items_pedido
        GROUP BY producto_id
    )
);
```

---

## Performance: JOIN vs Subconsulta

### Subconsulta

```sql
-- Más lento
SELECT nombre
FROM clientes
WHERE id IN (
    SELECT cliente_id
    FROM pedidos
);
```

### JOIN

```sql
-- Más rápido
SELECT DISTINCT c.nombre
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id;
```

**Regla general:**  
- **JOIN**: Preferible para relacionar tablas
- **Subconsulta**: Útil para cálculos y filtros complejos

---

## Subconsultas múltiples

```sql
-- Productos con precio entre mínimo y promedio
SELECT nombre, precio
FROM productos
WHERE precio >= (SELECT MIN(precio) FROM productos)
  AND precio <= (SELECT AVG(precio) FROM productos);
```

---

## Limitaciones en SQLite

### ❌ No soporta subconsultas en GROUP BY

```sql
-- ❌ ERROR en SQLite
SELECT 
    (SELECT nombre FROM categorias WHERE id = p.categoria_id) AS categoria,
    COUNT(*)
FROM productos p
GROUP BY categoria;
```

**Solución: Usar JOIN**
```sql
-- ✅ BIEN
SELECT c.nombre, COUNT(*)
FROM productos p
INNER JOIN categorias c ON p.categoria_id = c.id
GROUP BY c.id, c.nombre;
```

---

## Buenas prácticas

### ✅ Usar alias claros

```sql
-- BIEN
SELECT nombre
FROM clientes AS c
WHERE id IN (
    SELECT cliente_id
    FROM pedidos AS p
    WHERE p.total > 100
);
```

### ✅ Preferir EXISTS para existencia

```sql
-- Más eficiente
SELECT nombre
FROM clientes c
WHERE EXISTS (
    SELECT 1
    FROM pedidos p
    WHERE p.cliente_id = c.id
);

-- Menos eficiente
SELECT nombre
FROM clientes c
WHERE id IN (
    SELECT cliente_id
    FROM pedidos
);
```

### ✅ Evitar subconsultas correlacionadas si hay alternativa

```sql
-- ❌ Lento (correlacionada)
SELECT 
    nombre,
    (SELECT COUNT(*) FROM pedidos WHERE cliente_id = c.id) AS pedidos
FROM clientes c;

-- ✅ Rápido (JOIN + GROUP BY)
SELECT 
    c.nombre,
    COUNT(p.id) AS pedidos
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre;
```

---

## Errores comunes

### ❌ Subconsulta devuelve múltiples filas con =

```sql
-- ❌ ERROR: Subconsulta devuelve más de 1 fila
SELECT nombre
FROM productos
WHERE precio = (
    SELECT precio
    FROM productos
    WHERE categoria = 'Electrónica'  -- Devuelve 3 precios
);

-- ✅ BIEN: Usar IN
SELECT nombre
FROM productos
WHERE precio IN (
    SELECT precio
    FROM productos
    WHERE categoria = 'Electrónica'
);
```

### ❌ Usar columna de query externa sin correlación

```sql
-- ❌ ERROR
SELECT nombre
FROM clientes
WHERE (SELECT COUNT(*) FROM pedidos WHERE total > 100);

-- ✅ BIEN
SELECT nombre
FROM clientes c
WHERE (SELECT COUNT(*) FROM pedidos WHERE cliente_id = c.id AND total > 100) > 0;
```

---

## Comparación de técnicas

### Clientes sin pedidos

**1. LEFT JOIN:**
```sql
SELECT c.nombre
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
WHERE p.id IS NULL;
```

**2. NOT IN:**
```sql
SELECT nombre
FROM clientes
WHERE id NOT IN (
    SELECT cliente_id FROM pedidos
);
```

**3. NOT EXISTS:**
```sql
SELECT nombre
FROM clientes c
WHERE NOT EXISTS (
    SELECT 1 FROM pedidos WHERE cliente_id = c.id
);
```

**Recomendación:**  
- **NOT EXISTS**: Más rápido con grandes volúmenes
- **LEFT JOIN**: Más legible
- **NOT IN**: Cuidado con NULLs (no incluye NULL)

---

## Resumen

**Tipos de subconsultas:**
1. **En WHERE**: Filtrar filas (`IN`, `NOT IN`, `>`, `<`, `=`)
2. **En FROM**: Tabla derivada (alias obligatorio)
3. **En SELECT**: Columna calculada (devuelve 1 valor)
4. **Correlacionada**: Referencia query externa

**Operadores especiales:**
- `EXISTS` / `NOT EXISTS` - Verifica existencia
- `ANY` / `ALL` - Compara con conjunto
- `IN` / `NOT IN` - Pertenencia a conjunto

**Performance:**
- **JOIN** > **EXISTS** > **IN** > **Correlacionada**
- Evitar subconsultas correlacionadas si es posible
- Usar índices en columnas de subconsultas

**Próximo módulo:** PostgreSQL (instalación y configuración)

---

## Ejercicios

### Ejercicio 1

Productos con precio mayor al promedio de su categoría.

**Solución:**
```sql
SELECT nombre, categoria, precio
FROM productos p1
WHERE precio > (
    SELECT AVG(precio)
    FROM productos p2
    WHERE p2.categoria = p1.categoria
);
```

---

### Ejercicio 2

Clientes que han gastado más de $500.

**Solución:**
```sql
SELECT nombre
FROM clientes c
WHERE (SELECT SUM(total) FROM pedidos WHERE cliente_id = c.id) > 500;
```

---

### Ejercicio 3

Top 3 productos más vendidos.

**Solución:**
```sql
SELECT 
    nombre,
    (SELECT SUM(cantidad) 
     FROM items_pedido 
     WHERE producto_id = p.id) AS total_vendido
FROM productos p
ORDER BY total_vendido DESC
LIMIT 3;
```

---

### Ejercicio 4

Categorías con al menos un producto sin ventas.

**Solución:**
```sql
SELECT DISTINCT categoria
FROM productos p
WHERE EXISTS (
    SELECT 1
    FROM productos p2
    WHERE p2.categoria = p.categoria
      AND p2.id NOT IN (SELECT producto_id FROM items_pedido)
);
```

---

### Ejercicio 5

Empleados que ganan más que el promedio de todos los departamentos.

**Solución:**
```sql
SELECT nombre, departamento, salario
FROM empleados
WHERE salario > (
    SELECT AVG(salario) FROM empleados
);
```

---

## Recursos

- **[Subqueries Tutorial](https://www.sqlitetutorial.net/sqlite-subquery/)** - SQLite oficial
- **[EXISTS vs IN](https://www.sqlitetutorial.net/sqlite-exists/)** - Comparación
- **[Correlated Subqueries](https://www.w3schools.com/sql/sql_exists.asp)** - Ejemplos

¡Dominas **subconsultas**! Ahora tus queries pueden ser infinitamente más complejas. 🚀

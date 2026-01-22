# Funciones de Agregación en SQL

> **COUNT, SUM, AVG, MIN, MAX - Resumen y análisis de datos**

---

## ¿Qué son las funciones de agregación?

**Funciones de agregación** operan sobre **múltiples filas** y devuelven **un solo valor**.

**Ejemplos:**
- ¿Cuántos usuarios tengo? → `COUNT`
- ¿Cuál es el precio promedio? → `AVG`
- ¿Cuál es el total de ventas? → `SUM`

---

## Funciones básicas

```
Funciones de Agregación
├── COUNT()    Contar filas
├── SUM()      Sumar valores
├── AVG()      Promedio
├── MIN()      Valor mínimo
└── MAX()      Valor máximo
```

---

## COUNT() - Contar filas

Cuenta el número de filas.

### Sintaxis

```sql
COUNT(*)               -- Todas las filas
COUNT(columna)         -- Filas con valor no NULL en columna
COUNT(DISTINCT columna) -- Valores únicos
```

---

### COUNT(*) - Todas las filas

```sql
-- ¿Cuántos usuarios hay?
SELECT COUNT(*) FROM usuarios;
-- Resultado: 10
```

**Incluye filas con NULL.**

---

### COUNT(columna) - Filas no NULL

```sql
-- ¿Cuántos usuarios tienen email?
SELECT COUNT(email) FROM usuarios;
-- Resultado: 8 (2 usuarios tienen email NULL)
```

**Ignora NULL.**

---

### COUNT(DISTINCT columna) - Valores únicos

```sql
-- ¿Cuántas edades diferentes hay?
SELECT COUNT(DISTINCT edad) FROM usuarios;
-- Resultado: 5 (25, 28, 30, 35, 40)
```

---

### Ejemplo completo

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    email TEXT,
    edad INTEGER
);

INSERT INTO usuarios (id, nombre, email, edad) VALUES
    (1, 'Ana', 'ana@example.com', 25),
    (2, 'Bob', 'bob@example.com', 30),
    (3, 'Carlos', NULL, 25),
    (4, 'Diana', 'diana@example.com', 28),
    (5, 'Eva', NULL, 30);

-- Total de usuarios
SELECT COUNT(*) AS total FROM usuarios;
-- Resultado: 5

-- Usuarios con email
SELECT COUNT(email) AS con_email FROM usuarios;
-- Resultado: 3

-- Edades únicas
SELECT COUNT(DISTINCT edad) AS edades_diferentes FROM usuarios;
-- Resultado: 3 (25, 28, 30)
```

---

## SUM() - Sumar valores

Suma valores numéricos.

### Sintaxis

```sql
SUM(columna)
```

### Ejemplos

```sql
CREATE TABLE productos (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    precio REAL,
    stock INTEGER
);

INSERT INTO productos (id, nombre, precio, stock) VALUES
    (1, 'Laptop', 1000, 10),
    (2, 'Mouse', 20, 50),
    (3, 'Teclado', 80, 30);

-- Valor total del inventario (suma de precios)
SELECT SUM(precio) AS valor_total FROM productos;
-- Resultado: 1100

-- Stock total
SELECT SUM(stock) AS stock_total FROM productos;
-- Resultado: 90
```

---

### SUM con cálculos

```sql
-- Valor total considerando stock
SELECT SUM(precio * stock) AS valor_inventario FROM productos;
-- Resultado: (1000*10) + (20*50) + (80*30) = 10000 + 1000 + 2400 = 13400
```

---

### SUM con WHERE

```sql
-- Valor de productos con stock mayor a 20
SELECT SUM(precio) FROM productos WHERE stock > 20;
-- Resultado: 1080 (laptop + teclado)
```

---

## AVG() - Promedio

Calcula el promedio (media aritmética).

### Sintaxis

```sql
AVG(columna)
```

### Ejemplos

```sql
-- Edad promedio de usuarios
SELECT AVG(edad) FROM usuarios;
-- Resultado: 27.6 ((25+30+25+28+30)/5)

-- Precio promedio de productos
SELECT AVG(precio) FROM productos;
-- Resultado: 366.67 ((1000+20+80)/3)

-- Precio promedio de productos con stock > 20
SELECT AVG(precio) FROM productos WHERE stock > 20;
-- Resultado: 540 ((1000+80)/2)
```

---

### ROUND() con AVG

```sql
-- Promedio redondeado a 2 decimales
SELECT ROUND(AVG(precio), 2) AS precio_promedio FROM productos;
-- Resultado: 366.67
```

---

## MIN() - Valor mínimo

Encuentra el valor más pequeño.

### Sintaxis

```sql
MIN(columna)
```

### Ejemplos

```sql
-- Edad mínima
SELECT MIN(edad) FROM usuarios;
-- Resultado: 25

-- Producto más barato
SELECT MIN(precio) FROM productos;
-- Resultado: 20

-- Menor stock
SELECT MIN(stock) FROM productos;
-- Resultado: 10
```

---

### MIN con texto

```sql
-- Primer nombre alfabéticamente
SELECT MIN(nombre) FROM usuarios;
-- Resultado: 'Ana'
```

---

### MIN con fecha

```sql
CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente_id INTEGER,
    fecha DATE
);

INSERT INTO pedidos (id, cliente_id, fecha) VALUES
    (1, 1, '2024-01-15'),
    (2, 2, '2024-01-10'),
    (3, 1, '2024-01-20');

-- Pedido más antiguo
SELECT MIN(fecha) FROM pedidos;
-- Resultado: '2024-01-10'
```

---

## MAX() - Valor máximo

Encuentra el valor más grande.

### Sintaxis

```sql
MAX(columna)
```

### Ejemplos

```sql
-- Edad máxima
SELECT MAX(edad) FROM usuarios;
-- Resultado: 30

-- Producto más caro
SELECT MAX(precio) FROM productos;
-- Resultado: 1000

-- Mayor stock
SELECT MAX(stock) FROM productos;
-- Resultado: 50

-- Pedido más reciente
SELECT MAX(fecha) FROM pedidos;
-- Resultado: '2024-01-20'
```

---

## Múltiples funciones en una query

```sql
SELECT 
    COUNT(*) AS total_productos,
    SUM(precio) AS valor_total,
    AVG(precio) AS precio_promedio,
    MIN(precio) AS mas_barato,
    MAX(precio) AS mas_caro,
    SUM(stock) AS stock_total
FROM productos;
```

**Resultado:**
```
+-----------------+--------------+------------------+------------+-----------+-------------+
| total_productos | valor_total  | precio_promedio  | mas_barato | mas_caro  | stock_total |
+-----------------+--------------+------------------+------------+-----------+-------------+
|               3 |         1100 |           366.67 |         20 |      1000 |          90 |
+-----------------+--------------+------------------+------------+-----------+-------------+
```

---

## Agregación con WHERE

**WHERE filtra ANTES de agregar.**

```sql
-- Promedio de precio de productos con stock > 20
SELECT AVG(precio) FROM productos
WHERE stock > 20;
-- Resultado: 540 (laptop y teclado)

-- Total de stock de productos baratos (precio < 100)
SELECT SUM(stock) FROM productos
WHERE precio < 100;
-- Resultado: 80 (mouse + teclado)
```

---

## COALESCE() - Manejar NULL

**Devuelve el primer valor no NULL.**

```sql
-- Si precio es NULL, usar 0
SELECT 
    nombre,
    COALESCE(precio, 0) AS precio
FROM productos;
```

### Con agregaciones

```sql
-- Suma (tratar NULL como 0)
SELECT SUM(COALESCE(precio, 0)) FROM productos;

-- Promedio solo de valores no NULL (comportamiento por defecto)
SELECT AVG(precio) FROM productos;
```

---

## Caso práctico: Estadísticas de ventas

### Tablas

```sql
CREATE TABLE clientes (
    id INTEGER PRIMARY KEY,
    nombre TEXT
);

CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente_id INTEGER,
    fecha DATE,
    total REAL,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

-- Datos
INSERT INTO clientes VALUES (1, 'Ana'), (2, 'Bob'), (3, 'Carlos');

INSERT INTO pedidos (id, cliente_id, fecha, total) VALUES
    (1, 1, '2024-01-15', 1200),
    (2, 1, '2024-01-20', 50),
    (3, 2, '2024-01-18', 300),
    (4, 2, '2024-01-22', 150),
    (5, 3, '2024-01-25', 800);
```

---

### Queries analíticas

**1. Estadísticas generales:**

```sql
SELECT 
    COUNT(*) AS total_pedidos,
    SUM(total) AS ingresos_totales,
    AVG(total) AS ticket_promedio,
    MIN(total) AS pedido_menor,
    MAX(total) AS pedido_mayor
FROM pedidos;
```

**Resultado:**
```
+---------------+------------------+-----------------+--------------+--------------+
| total_pedidos | ingresos_totales | ticket_promedio | pedido_menor | pedido_mayor |
+---------------+------------------+-----------------+--------------+--------------+
|             5 |             2500 |             500 |           50 |         1200 |
+---------------+------------------+-----------------+--------------+--------------+
```

---

**2. Pedidos por cliente:**

```sql
SELECT 
    c.nombre,
    COUNT(p.id) AS num_pedidos,
    SUM(p.total) AS total_gastado,
    AVG(p.total) AS ticket_promedio
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre;
```

**Resultado:**
```
+---------+-------------+---------------+-----------------+
| nombre  | num_pedidos | total_gastado | ticket_promedio |
+---------+-------------+---------------+-----------------+
| Ana     |           2 |          1250 |             625 |
| Bob     |           2 |           450 |             225 |
| Carlos  |           1 |           800 |             800 |
+---------+-------------+---------------+-----------------+
```

---

**3. Pedidos en enero 2024:**

```sql
SELECT 
    COUNT(*) AS pedidos_enero,
    SUM(total) AS ingresos_enero
FROM pedidos
WHERE fecha BETWEEN '2024-01-01' AND '2024-01-31';
```

---

**4. Cliente con mayor gasto:**

```sql
SELECT 
    c.nombre,
    SUM(p.total) AS total_gastado
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre
ORDER BY total_gastado DESC
LIMIT 1;
```

**Resultado:**
```
+---------+---------------+
| nombre  | total_gastado |
+---------+---------------+
| Ana     |          1250 |
+---------+---------------+
```

---

## Funciones adicionales

### LENGTH() - Longitud de texto

```sql
-- Promedio de longitud de nombres
SELECT AVG(LENGTH(nombre)) FROM usuarios;
-- Resultado: 4.8
```

---

### DISTINCT dentro de agregaciones

```sql
-- Número de clientes que han hecho pedidos
SELECT COUNT(DISTINCT cliente_id) FROM pedidos;
-- Resultado: 3

-- Suma de precios únicos
SELECT SUM(DISTINCT precio) FROM productos;
-- Si hay precios duplicados, solo suma una vez cada valor único
```

---

## Comparación: Python vs SQL

### Contar

**Python:**
```python
usuarios = [...]
total = len(usuarios)
```

**SQL:**
```sql
SELECT COUNT(*) FROM usuarios;
```

---

### Sumar

**Python:**
```python
total = sum(p['precio'] for p in productos)
```

**SQL:**
```sql
SELECT SUM(precio) FROM productos;
```

---

### Promedio

**Python:**
```python
from statistics import mean
promedio = mean(u['edad'] for u in usuarios)
```

**SQL:**
```sql
SELECT AVG(edad) FROM usuarios;
```

---

### Mínimo/Máximo

**Python:**
```python
min_edad = min(u['edad'] for u in usuarios)
max_edad = max(u['edad'] for u in usuarios)
```

**SQL:**
```sql
SELECT MIN(edad), MAX(edad) FROM usuarios;
```

---

## Buenas prácticas

### ✅ Usar alias descriptivos

```sql
-- BIEN
SELECT 
    COUNT(*) AS total_productos,
    AVG(precio) AS precio_promedio
FROM productos;

-- Evitar
SELECT COUNT(*), AVG(precio) FROM productos;
```

### ✅ ROUND() para decimales

```sql
-- BIEN
SELECT ROUND(AVG(precio), 2) AS precio_promedio FROM productos;

-- Evitar (muchos decimales)
SELECT AVG(precio) FROM productos;
```

### ✅ Manejar NULL

```sql
-- Explícito con NULL
SELECT AVG(COALESCE(precio, 0)) FROM productos;
```

### ✅ Combinar con WHERE para filtrar

```sql
SELECT AVG(precio) 
FROM productos
WHERE activo = 1;  -- solo productos activos
```

---

## Errores comunes

### ❌ Mezclar columnas normales con agregaciones

```sql
-- ❌ ERROR
SELECT nombre, COUNT(*) FROM usuarios;
-- ¿Qué nombre mostrar si hay múltiples usuarios?

-- ✅ BIEN (con GROUP BY, próximo módulo)
SELECT nombre, COUNT(*) FROM usuarios GROUP BY nombre;
```

### ❌ COUNT(*) vs COUNT(columna)

```sql
-- COUNT(*) cuenta todas las filas
SELECT COUNT(*) FROM usuarios;  -- 5 usuarios

-- COUNT(columna) ignora NULL
SELECT COUNT(email) FROM usuarios;  -- 3 (2 tienen NULL)
```

### ❌ Usar agregaciones en WHERE

```sql
-- ❌ ERROR
SELECT * FROM productos
WHERE SUM(precio) > 1000;

-- ✅ BIEN (usar HAVING con GROUP BY)
SELECT categoria, SUM(precio)
FROM productos
GROUP BY categoria
HAVING SUM(precio) > 1000;
```

---

## Resumen

**Funciones de agregación:**
- `COUNT(*)` - Contar todas las filas
- `COUNT(columna)` - Contar no NULL
- `COUNT(DISTINCT columna)` - Contar únicos
- `SUM(columna)` - Sumar
- `AVG(columna)` - Promedio
- `MIN(columna)` - Mínimo
- `MAX(columna)` - Máximo

**Características:**
- Operan sobre múltiples filas
- Devuelven un solo valor
- Ignoran NULL (excepto COUNT(*))
- Se combinan con WHERE para filtrar

**Sintaxis:**
```sql
SELECT 
    COUNT(*) AS total,
    AVG(columna) AS promedio,
    SUM(columna) AS suma
FROM tabla
WHERE condicion;
```

**Próximo módulo:** GROUP BY y HAVING (agregar por grupos)

---

## Ejercicios

### Ejercicio 1

Contar total de productos y stock total.

**Solución:**
```sql
SELECT 
    COUNT(*) AS total_productos,
    SUM(stock) AS stock_total
FROM productos;
```

---

### Ejercicio 2

Precio promedio de productos con stock > 10.

**Solución:**
```sql
SELECT AVG(precio) AS precio_promedio
FROM productos
WHERE stock > 10;
```

---

### Ejercicio 3

Producto más caro y más barato.

**Solución:**
```sql
SELECT 
    MAX(precio) AS mas_caro,
    MIN(precio) AS mas_barato
FROM productos;
```

---

### Ejercicio 4

Total de ventas (suma de todos los pedidos).

**Solución:**
```sql
SELECT SUM(total) AS ventas_totales
FROM pedidos;
```

---

### Ejercicio 5

Número de clientes que han hecho al menos un pedido.

**Solución:**
```sql
SELECT COUNT(DISTINCT cliente_id) AS clientes_activos
FROM pedidos;
```

---

## Recursos

- **[Aggregate Functions](https://www.sqlitetutorial.net/sqlite-aggregate-functions/)** - Tutorial completo
- **[COUNT vs COUNT(*)](https://www.sqlitetutorial.net/sqlite-count-function/)** - Diferencias
- **[AVG Function](https://www.sqlitetutorial.net/sqlite-avg/)** - Promedio

Ahora dominas las **funciones de agregación**. 📊

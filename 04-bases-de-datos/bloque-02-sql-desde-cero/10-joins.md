# JOINs en SQL

> **Relacionar tablas y combinar datos**

---

## ¿Qué son los JOINs?

**JOIN** combina filas de dos o más tablas basándose en una relación entre ellas.

**Analogía:**  
Como unir dos hojas de Excel usando una columna común (ej: ID de usuario).

---

## Tipos de JOINs

```
JOINs
├── INNER JOIN    (Solo coincidencias)
├── LEFT JOIN     (Todos de izquierda + coincidencias)
├── RIGHT JOIN    (Todos de derecha + coincidencias)
├── FULL JOIN     (Todos de ambas)
└── CROSS JOIN    (Producto cartesiano)
```

---

## Escenario de ejemplo

Dos tablas relacionadas:

```sql
-- Tabla: usuarios
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre TEXT NOT NULL
);

INSERT INTO usuarios (id, nombre) VALUES
    (1, 'Ana'),
    (2, 'Bob'),
    (3, 'Carlos'),
    (4, 'Diana');

-- Tabla: posts
CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    usuario_id INTEGER,
    titulo TEXT,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);

INSERT INTO posts (id, usuario_id, titulo) VALUES
    (1, 1, 'Post de Ana'),
    (2, 1, 'Otro post de Ana'),
    (3, 2, 'Post de Bob'),
    (4, NULL, 'Post sin autor');  -- sin usuario
```

**Estado de las tablas:**

```
usuarios                    posts
+----+---------+            +----+------------+------------------+
| id | nombre  |            | id | usuario_id | titulo           |
+----+---------+            +----+------------+------------------+
|  1 | Ana     |            |  1 |          1 | Post de Ana      |
|  2 | Bob     |            |  2 |          1 | Otro post de Ana |
|  3 | Carlos  |            |  3 |          2 | Post de Bob      |
|  4 | Diana   |            |  4 |       NULL | Post sin autor   |
+----+---------+            +----+------------+------------------+
    ↑                                ↑
  tiene posts                   sin posts (Carlos, Diana)
                                post sin usuario (NULL)
```

---

## INNER JOIN

**Devuelve solo filas donde hay coincidencia en ambas tablas.**

### Sintaxis

```sql
SELECT columnas
FROM tabla1
INNER JOIN tabla2 ON tabla1.columna = tabla2.columna;
```

### Ejemplo

```sql
SELECT usuarios.nombre, posts.titulo
FROM usuarios
INNER JOIN posts ON usuarios.id = posts.usuario_id;
```

**Resultado:**
```
+---------+------------------+
| nombre  | titulo           |
+---------+------------------+
| Ana     | Post de Ana      | ← usuario 1
| Ana     | Otro post de Ana | ← usuario 1
| Bob     | Post de Bob      | ← usuario 2
+---------+------------------+
```

**Observa:**
- ✅ Ana aparece 2 veces (tiene 2 posts)
- ✅ Bob aparece 1 vez (tiene 1 post)
- ❌ Carlos NO aparece (sin posts)
- ❌ Diana NO aparece (sin posts)
- ❌ Post sin autor NO aparece (usuario_id = NULL)

---

### Diagrama de Venn

```
    usuarios         posts
    ┌─────────┐ ┌─────────┐
    │         │ │         │
    │    ┌────┼─┼────┐    │
    │    │ INNER  │    │
    │    │  JOIN  │    │
    │    └────┼─┼────┘    │
    │         │ │         │
    └─────────┘ └─────────┘
         ↑
    Solo esta parte
```

---

### Alias de tablas

**Para código más corto:**

```sql
SELECT u.nombre, p.titulo
FROM usuarios AS u
INNER JOIN posts AS p ON u.id = p.usuario_id;

-- Sin AS (también válido)
SELECT u.nombre, p.titulo
FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;
```

---

## LEFT JOIN (LEFT OUTER JOIN)

**Devuelve todas las filas de la tabla izquierda + coincidencias de la derecha.**

### Sintaxis

```sql
SELECT columnas
FROM tabla1
LEFT JOIN tabla2 ON tabla1.columna = tabla2.columna;
```

### Ejemplo

```sql
SELECT u.nombre, p.titulo
FROM usuarios u
LEFT JOIN posts p ON u.id = p.usuario_id;
```

**Resultado:**
```
+---------+------------------+
| nombre  | titulo           |
+---------+------------------+
| Ana     | Post de Ana      | ← tiene posts
| Ana     | Otro post de Ana | ← tiene posts
| Bob     | Post de Bob      | ← tiene posts
| Carlos  | NULL             | ← sin posts
| Diana   | NULL             | ← sin posts
+---------+------------------+
```

**Observa:**
- ✅ Todos los usuarios aparecen
- ✅ Usuarios sin posts tienen `NULL` en columnas de posts

---

### Diagrama de Venn

```
    usuarios         posts
    ┌─────────┐ ┌─────────┐
    │█████████│ │         │
    │█████████│ │         │
    │██████┌──┼─┼────┐    │
    │██████│LEFT  │    │
    │██████│ JOIN │    │
    │██████└──┼─┼────┘    │
    │█████████│ │         │
    └─────────┘ └─────────┘
         ↑
   Toda la izquierda
```

---

### Uso común: Encontrar registros sin relación

```sql
-- Usuarios sin posts
SELECT u.nombre
FROM usuarios u
LEFT JOIN posts p ON u.id = p.usuario_id
WHERE p.id IS NULL;
```

**Resultado:**
```
+---------+
| nombre  |
+---------+
| Carlos  |
| Diana   |
+---------+
```

**Lógica:**
1. LEFT JOIN trae todos los usuarios
2. WHERE p.id IS NULL filtra solo usuarios sin posts

---

## RIGHT JOIN (RIGHT OUTER JOIN)

**Devuelve todas las filas de la tabla derecha + coincidencias de la izquierda.**

### Sintaxis

```sql
SELECT columnas
FROM tabla1
RIGHT JOIN tabla2 ON tabla1.columna = tabla2.columna;
```

**⚠️ SQLite NO soporta RIGHT JOIN.**

**Solución:** Invertir tablas con LEFT JOIN.

```sql
-- RIGHT JOIN (no funciona en SQLite)
SELECT u.nombre, p.titulo
FROM usuarios u
RIGHT JOIN posts p ON u.id = p.usuario_id;

-- Equivalente con LEFT JOIN
SELECT u.nombre, p.titulo
FROM posts p
LEFT JOIN usuarios u ON p.usuario_id = u.id;
```

**Resultado:**
```
+---------+------------------+
| nombre  | titulo           |
+---------+------------------+
| Ana     | Post de Ana      |
| Ana     | Otro post de Ana |
| Bob     | Post de Bob      |
| NULL    | Post sin autor   | ← post sin usuario
+---------+------------------+
```

---

### Diagrama de Venn

```
    usuarios         posts
    ┌─────────┐ ┌─────────┐
    │         │ │█████████│
    │         │ │█████████│
    │    ┌────┼─┼──██████ │
    │    │ RIGHT │██████ │
    │    │  JOIN │██████ │
    │    └────┼─┼──██████ │
    │         │ │█████████│
    └─────────┘ └─────────┘
              ↑
      Toda la derecha
```

---

## FULL OUTER JOIN

**Devuelve todas las filas de ambas tablas (coincidan o no).**

**⚠️ SQLite NO soporta FULL OUTER JOIN.**

**Solución:** Combinar LEFT JOIN y UNION.

```sql
-- Simular FULL OUTER JOIN
SELECT u.nombre, p.titulo
FROM usuarios u
LEFT JOIN posts p ON u.id = p.usuario_id

UNION

SELECT u.nombre, p.titulo
FROM posts p
LEFT JOIN usuarios u ON p.usuario_id = u.id
WHERE u.id IS NULL;
```

**Resultado:**
```
+---------+------------------+
| nombre  | titulo           |
+---------+------------------+
| Ana     | Post de Ana      |
| Ana     | Otro post de Ana |
| Bob     | Post de Bob      |
| Carlos  | NULL             | ← usuario sin posts
| Diana   | NULL             | ← usuario sin posts
| NULL    | Post sin autor   | ← post sin usuario
+---------+------------------+
```

---

### Diagrama de Venn

```
    usuarios         posts
    ┌─────────┐ ┌─────────┐
    │█████████│ │█████████│
    │█████████│ │█████████│
    │██████┌──┼─┼────██████│
    │██████│ FULL │██████│
    │██████│ JOIN │██████│
    │██████└──┼─┼────██████│
    │█████████│ │█████████│
    └─────────┘ └─────────┘
         ↑
      Todo
```

---

## CROSS JOIN

**Producto cartesiano: combina cada fila de tabla1 con cada fila de tabla2.**

### Sintaxis

```sql
SELECT columnas
FROM tabla1
CROSS JOIN tabla2;
```

### Ejemplo

```sql
SELECT u.nombre, p.titulo
FROM usuarios u
CROSS JOIN posts p;
```

**Resultado:**
```
+---------+------------------+
| nombre  | titulo           |
+---------+------------------+
| Ana     | Post de Ana      | ← Ana con cada post
| Ana     | Otro post de Ana |
| Ana     | Post de Bob      |
| Ana     | Post sin autor   |
| Bob     | Post de Ana      | ← Bob con cada post
| Bob     | Otro post de Ana |
| Bob     | Post de Bob      |
| Bob     | Post sin autor   |
| Carlos  | Post de Ana      | ← Carlos con cada post
| Carlos  | Otro post de Ana |
| Carlos  | Post de Bob      |
| Carlos  | Post sin autor   |
| Diana   | Post de Ana      | ← Diana con cada post
| Diana   | Otro post de Ana |
| Diana   | Post de Bob      |
| Diana   | Post sin autor   |
+---------+------------------+
```

**Total:** 4 usuarios × 4 posts = 16 filas

**Uso común:** Generar todas las combinaciones.

---

## Múltiples JOINs

Unir más de 2 tablas.

### Escenario: Blog completo

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

CREATE TABLE comentarios (
    id INTEGER PRIMARY KEY,
    post_id INTEGER,
    usuario_id INTEGER,
    texto TEXT,
    FOREIGN KEY (post_id) REFERENCES posts(id),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);

-- Datos
INSERT INTO usuarios (id, nombre) VALUES (1, 'Ana'), (2, 'Bob');

INSERT INTO posts (id, usuario_id, titulo) VALUES
    (1, 1, 'Post de Ana'),
    (2, 2, 'Post de Bob');

INSERT INTO comentarios (id, post_id, usuario_id, texto) VALUES
    (1, 1, 2, 'Bob comenta en post de Ana'),
    (2, 1, 1, 'Ana responde su propio post'),
    (3, 2, 1, 'Ana comenta en post de Bob');
```

---

### Query con múltiples JOINs

```sql
SELECT 
    p.titulo AS post,
    u_post.nombre AS autor_post,
    c.texto AS comentario,
    u_com.nombre AS autor_comentario
FROM posts p
INNER JOIN usuarios u_post ON p.usuario_id = u_post.id
LEFT JOIN comentarios c ON p.id = c.post_id
LEFT JOIN usuarios u_com ON c.usuario_id = u_com.id;
```

**Resultado:**
```
+---------------+------------+-------------------------------+-----------------+
| post          | autor_post | comentario                    | autor_comentario|
+---------------+------------+-------------------------------+-----------------+
| Post de Ana   | Ana        | Bob comenta en post de Ana    | Bob             |
| Post de Ana   | Ana        | Ana responde su propio post   | Ana             |
| Post de Bob   | Bob        | Ana comenta en post de Bob    | Ana             |
+---------------+------------+-------------------------------+-----------------+
```

---

## Self JOIN

Unir tabla consigo misma.

### Escenario: Empleados y jefes

```sql
CREATE TABLE empleados (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    jefe_id INTEGER,
    FOREIGN KEY (jefe_id) REFERENCES empleados(id)
);

INSERT INTO empleados (id, nombre, jefe_id) VALUES
    (1, 'CEO', NULL),
    (2, 'Manager', 1),
    (3, 'Empleado 1', 2),
    (4, 'Empleado 2', 2);
```

### Query

```sql
SELECT 
    e.nombre AS empleado,
    j.nombre AS jefe
FROM empleados e
LEFT JOIN empleados j ON e.jefe_id = j.id;
```

**Resultado:**
```
+-------------+----------+
| empleado    | jefe     |
+-------------+----------+
| CEO         | NULL     | ← sin jefe
| Manager     | CEO      |
| Empleado 1  | Manager  |
| Empleado 2  | Manager  |
+-------------+----------+
```

---

## Ejemplo práctico: E-commerce

### Tablas

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

CREATE TABLE clientes (
    id INTEGER PRIMARY KEY,
    nombre TEXT,
    email TEXT
);

CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente_id INTEGER,
    fecha DATE,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
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
INSERT INTO categorias VALUES (1, 'Electrónica'), (2, 'Ropa');
INSERT INTO productos VALUES 
    (1, 'Laptop', 1, 1000),
    (2, 'Mouse', 1, 20),
    (3, 'Camisa', 2, 30);
INSERT INTO clientes VALUES (1, 'Ana', 'ana@example.com');
INSERT INTO pedidos VALUES (1, 1, '2024-01-15');
INSERT INTO items_pedido VALUES 
    (1, 1, 1, 1),  -- 1 laptop
    (2, 1, 2, 2);  -- 2 mouse
```

---

### Queries complejas

**1. Pedidos con detalle completo:**

```sql
SELECT 
    c.nombre AS cliente,
    c.email,
    ped.fecha,
    prod.nombre AS producto,
    cat.nombre AS categoria,
    i.cantidad,
    prod.precio,
    i.cantidad * prod.precio AS subtotal
FROM pedidos ped
INNER JOIN clientes c ON ped.cliente_id = c.id
INNER JOIN items_pedido i ON ped.id = i.pedido_id
INNER JOIN productos prod ON i.producto_id = prod.id
INNER JOIN categorias cat ON prod.categoria_id = cat.id;
```

**Resultado:**
```
+---------+------------------+------------+----------+--------------+----------+--------+-----------+
| cliente | email            | fecha      | producto | categoria    | cantidad | precio | subtotal  |
+---------+------------------+------------+----------+--------------+----------+--------+-----------+
| Ana     | ana@example.com  | 2024-01-15 | Laptop   | Electrónica  |        1 |   1000 |      1000 |
| Ana     | ana@example.com  | 2024-01-15 | Mouse    | Electrónica  |        2 |     20 |        40 |
+---------+------------------+------------+----------+--------------+----------+--------+-----------+
```

---

**2. Productos más vendidos:**

```sql
SELECT 
    p.nombre,
    SUM(i.cantidad) AS total_vendido
FROM productos p
LEFT JOIN items_pedido i ON p.id = i.producto_id
GROUP BY p.id, p.nombre
ORDER BY total_vendido DESC;
```

---

**3. Clientes sin pedidos:**

```sql
SELECT c.nombre, c.email
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
WHERE p.id IS NULL;
```

---

## Comparación de JOINs

### Tabla resumen

| JOIN | Descripción | Filas devueltas |
|------|-------------|-----------------|
| **INNER** | Solo coincidencias | Solo donde hay match |
| **LEFT** | Todos de izquierda | Izquierda + matches |
| **RIGHT** | Todos de derecha | Derecha + matches |
| **FULL** | Todos de ambas | Todas las filas |
| **CROSS** | Producto cartesiano | n × m filas |

---

### Ejemplo comparativo

```sql
-- Datos:
-- usuarios: Ana (1), Bob (2), Carlos (3)
-- posts: Post 1 (usuario 1), Post 2 (usuario 1), Post 3 (usuario NULL)

-- INNER JOIN: 2 filas (solo Ana con posts)
SELECT u.nombre, p.titulo
FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;

-- LEFT JOIN: 4 filas (todos los usuarios)
SELECT u.nombre, p.titulo
FROM usuarios u
LEFT JOIN posts p ON u.id = p.usuario_id;

-- RIGHT JOIN (como LEFT invertido): 3 filas (todos los posts)
SELECT u.nombre, p.titulo
FROM posts p
LEFT JOIN usuarios u ON p.usuario_id = u.id;

-- CROSS JOIN: 9 filas (3 usuarios × 3 posts)
SELECT u.nombre, p.titulo
FROM usuarios u
CROSS JOIN posts p;
```

---

## Buenas prácticas

### ✅ Usar alias claros

```sql
-- BIEN
SELECT u.nombre, p.titulo
FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;

-- Evitar
SELECT usuarios.nombre, posts.titulo
FROM usuarios
INNER JOIN posts ON usuarios.id = posts.usuario_id;
```

### ✅ Especificar columnas explícitamente

```sql
-- BIEN
SELECT u.nombre, p.titulo
FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;

-- Evitar (ambiguo con columnas duplicadas)
SELECT *
FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;
```

### ✅ LEFT JOIN para encontrar registros sin relación

```sql
SELECT u.nombre
FROM usuarios u
LEFT JOIN posts p ON u.id = p.usuario_id
WHERE p.id IS NULL;
```

### ✅ Índices en foreign keys

```sql
CREATE INDEX idx_posts_usuario ON posts(usuario_id);
-- Acelera los JOINs
```

---

## Errores comunes

### ❌ Olvidar condición de JOIN

```sql
-- MAL (producto cartesiano accidental)
SELECT u.nombre, p.titulo
FROM usuarios u, posts p;

-- BIEN
SELECT u.nombre, p.titulo
FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;
```

### ❌ Comparar NULL con =

```sql
-- MAL
SELECT * FROM posts WHERE usuario_id = NULL;

-- BIEN
SELECT * FROM posts WHERE usuario_id IS NULL;
```

### ❌ Ambigüedad en columnas

```sql
-- MAL (si ambas tablas tienen columna "id")
SELECT id FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;

-- BIEN
SELECT u.id FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;
```

---

## Resumen

**JOINs combinan tablas:**
- **INNER JOIN** - Solo coincidencias
- **LEFT JOIN** - Todos de izquierda + coincidencias
- **RIGHT JOIN** - Todos de derecha + coincidencias (invertir con LEFT)
- **FULL JOIN** - Todos de ambas (usar UNION en SQLite)
- **CROSS JOIN** - Todas las combinaciones

**Sintaxis:**
```sql
SELECT columnas
FROM tabla1
[INNER|LEFT|RIGHT|FULL|CROSS] JOIN tabla2
ON tabla1.columna = tabla2.columna;
```

**Múltiples JOINs:**
```sql
FROM tabla1
JOIN tabla2 ON ...
JOIN tabla3 ON ...
JOIN tabla4 ON ...
```

**Próximo módulo:** Funciones de agregación (COUNT, SUM, AVG)

---

## Ejercicios

### Ejercicio 1

Mostrar usuarios con sus posts (solo usuarios con posts).

**Solución:**
```sql
SELECT u.nombre, p.titulo
FROM usuarios u
INNER JOIN posts p ON u.id = p.usuario_id;
```

---

### Ejercicio 2

Mostrar todos los usuarios y sus posts (incluir usuarios sin posts).

**Solución:**
```sql
SELECT u.nombre, p.titulo
FROM usuarios u
LEFT JOIN posts p ON u.id = p.usuario_id;
```

---

### Ejercicio 3

Encontrar usuarios sin posts.

**Solución:**
```sql
SELECT u.nombre
FROM usuarios u
LEFT JOIN posts p ON u.id = p.usuario_id
WHERE p.id IS NULL;
```

---

### Ejercicio 4

Mostrar pedidos con cliente y productos (múltiples JOINs).

**Solución:**
```sql
SELECT 
    c.nombre AS cliente,
    ped.fecha,
    prod.nombre AS producto,
    i.cantidad
FROM pedidos ped
INNER JOIN clientes c ON ped.cliente_id = c.id
INNER JOIN items_pedido i ON ped.id = i.pedido_id
INNER JOIN productos prod ON i.producto_id = prod.id;
```

---

## Recursos

- **[SQL Joins Visualizer](https://sql-joins.leopard.in.ua/)** - Visualizar JOINs
- **[Joins Explained](https://www.w3schools.com/sql/sql_join.asp)** - Tutorial completo
- **[SQLite Joins](https://www.sqlitetutorial.net/sqlite-join/)** - Guía SQLite

Ahora dominas los **JOINs en SQL**. 🔗

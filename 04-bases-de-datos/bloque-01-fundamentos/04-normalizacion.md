# Normalización de Bases de Datos

> **Cómo organizar datos para evitar redundancia y anomalías**

---

## ¿Qué es la Normalización?

**Definición:**  
La **normalización** es el proceso de organizar datos en una base de datos para:
- ✅ Eliminar redundancia (duplicación)
- ✅ Evitar anomalías (errores de inserción, actualización, eliminación)
- ✅ Garantizar integridad
- ✅ Facilitar mantenimiento

**Analogía:**  
Como organizar tu armario:
- ❌ Ropa amontonada (difícil encontrar, duplicados)
- ✅ Ropa por categorías (fácil acceso, sin duplicados)

---

## Problemas que Resuelve

### Problema 1: Redundancia (Duplicación)

**Base de datos NO normalizada:**

```
Tabla: pedidos
+----------+--------------+------------------+----------------------+
| pedido_id| producto     | cliente_nombre   | cliente_email        |
+----------+--------------+------------------+----------------------+
|        1 | Laptop       | Ana García       | ana@example.com      |
|        2 | Mouse        | Ana García       | ana@example.com      |
|        3 | Teclado      | Bob Smith        | bob@example.com      |
+----------+--------------+------------------+----------------------+
          ↑ Datos de Ana duplicados ↑
```

**Problemas:**
- ❌ Nombre/email de Ana aparece 2 veces
- ❌ Ocupa más espacio
- ❌ ¿Qué pasa si Ana cambia su email?

---

### Problema 2: Anomalías

#### Anomalía de Actualización

```
-- Ana cambia su email
UPDATE pedidos SET cliente_email = 'ana_nueva@example.com'
WHERE cliente_nombre = 'Ana García';

-- Si olvidas actualizar una fila, datos inconsistentes:
+----------+--------------+-----------------+------------------------+
| pedido_id| producto     | cliente_nombre  | cliente_email          |
+----------+--------------+-----------------+------------------------+
|        1 | Laptop       | Ana García      | ana_nueva@example.com |
|        2 | Mouse        | Ana García      | ana@example.com       | ← ❌
+----------+--------------+-----------------+------------------------+
```

#### Anomalía de Inserción

```
-- No puedes agregar un cliente sin pedido
INSERT INTO pedidos (producto, cliente_nombre, cliente_email)
VALUES (NULL, 'Carlos', 'carlos@example.com');  -- ❌ producto es NULL?
```

#### Anomalía de Eliminación

```
-- Si eliminas el último pedido de Bob, pierdes su info
DELETE FROM pedidos WHERE pedido_id = 3;
-- Ahora no sabes que Bob existe
```

---

## Proceso de Normalización

La normalización se realiza en **formas normales** (NF):

```
Datos sin normalizar
    ↓
Primera Forma Normal (1NF)
    ↓
Segunda Forma Normal (2NF)
    ↓
Tercera Forma Normal (3NF)
    ↓
Forma Normal de Boyce-Codd (BCNF)  ← generalmente hasta aquí
```

---

## 1. Primera Forma Normal (1NF)

**Reglas:**
1. ✅ Cada columna debe contener **valores atómicos** (indivisibles)
2. ✅ Sin grupos repetidos
3. ✅ Cada fila debe ser única (tener clave primaria)

### Ejemplo 1: Valores atómicos

**❌ NO normalizado:**

```
Tabla: usuarios
+----+-----------+---------------------------+
| id | nombre    | telefonos                 |
+----+-----------+---------------------------+
|  1 | Ana       | 555-1234, 555-5678        | ← múltiples valores
|  2 | Bob       | 555-9999                  |
+----+-----------+---------------------------+
```

**Problema:** `telefonos` no es atómico (tiene 2 valores).

**✅ 1NF:**

```
Tabla: usuarios                    Tabla: telefonos
+----+-----------+                 +----+------------+-----------+
| id | nombre    |                 | id | usuario_id | telefono  |
+----+-----------+                 +----+------------+-----------+
|  1 | Ana       |                 |  1 |          1 | 555-1234  |
|  2 | Bob       |                 |  2 |          1 | 555-5678  |
+----+-----------+                 |  3 |          2 | 555-9999  |
                                   +----+------------+-----------+
```

---

### Ejemplo 2: Sin grupos repetidos

**❌ NO normalizado:**

```
Tabla: pedidos
+----------+-----------+-----------+-----------+
| pedido_id| producto1 | producto2 | producto3 |
+----------+-----------+-----------+-----------+
|        1 | Laptop    | Mouse     | Teclado   |
|        2 | Monitor   | NULL      | NULL      |
+----------+-----------+-----------+-----------+
```

**Problemas:**
- ❌ Columnas repetidas (producto1, producto2, ...)
- ❌ Límite de productos (¿y si hay 4?)
- ❌ Muchos NULL

**✅ 1NF:**

```
Tabla: pedidos                     Tabla: items_pedido
+----------+                       +----------+------------+
| pedido_id|                       | pedido_id| producto   |
+----------+                       +----------+------------+
|        1 |                       |        1 | Laptop     |
|        2 |                       |        1 | Mouse      |
+----------+                       |        1 | Teclado    |
                                   |        2 | Monitor    |
                                   +----------+------------+
```

---

### Ejemplo 3: Clave primaria

**❌ NO normalizado:**

```
Tabla: usuarios
+-----------+-------------------+
| nombre    | email             |
+-----------+-------------------+
| Ana       | ana@example.com   |
| Ana       | ana2@example.com  | ← duplicado!
+-----------+-------------------+
```

**✅ 1NF:**

```
Tabla: usuarios
+----+-----------+-------------------+
| id | nombre    | email             |  ← id es PRIMARY KEY
+----+-----------+-------------------+
|  1 | Ana       | ana@example.com   |
|  2 | Ana       | ana2@example.com  |
+----+-----------+-------------------+
```

---

## 2. Segunda Forma Normal (2NF)

**Requisitos:**
1. ✅ Cumple 1NF
2. ✅ No hay **dependencias parciales** de la clave primaria

**Dependencia parcial:**  
Cuando un atributo depende solo de **parte** de la clave primaria compuesta.

### Ejemplo

**❌ NO está en 2NF:**

```
Tabla: items_pedido
+-----------+-------------+------------------+----------------+
| pedido_id | producto_id | cliente_nombre   | precio_unitario|
+-----------+-------------+------------------+----------------+
|         1 |         100 | Ana García       |           1000 |
|         1 |         200 | Ana García       |             50 |
|         2 |         100 | Bob Smith        |           1000 |
+-----------+-------------+------------------+----------------+
     ↑            ↑             ↑
PRIMARY KEY (ambas)    Depende solo de pedido_id ← ❌ dependencia parcial
```

**Problema:**  
`cliente_nombre` depende solo de `pedido_id`, no de la clave completa `(pedido_id, producto_id)`.

**✅ 2NF:**

```
Tabla: pedidos                     Tabla: items_pedido
+-----------+-----------------+    +-----------+-------------+----------------+
| pedido_id | cliente_nombre  |    | pedido_id | producto_id | precio_unitario|
+-----------+-----------------+    +-----------+-------------+----------------+
|         1 | Ana García      |    |         1 |         100 |           1000 |
|         2 | Bob Smith       |    |         1 |         200 |             50 |
+-----------+-----------------+    |         2 |         100 |           1000 |
                                   +-----------+-------------+----------------+
```

Ahora `cliente_nombre` está en la tabla correcta.

---

### Otro Ejemplo

**❌ NO está en 2NF:**

```
Tabla: inscripciones
+----------------+----------+-------------------+
| estudiante_id  | curso_id | profesor_nombre   |
+----------------+----------+-------------------+
|              1 |      101 | Dr. López         |
|              2 |      101 | Dr. López         |
|              1 |      102 | Dra. Martínez     |
+----------------+----------+-------------------+
     PRIMARY KEY (ambas)      ↑
                         Depende solo de curso_id ← ❌
```

**✅ 2NF:**

```
Tabla: inscripciones           Tabla: cursos
+----------------+----------+   +----------+-------------------+
| estudiante_id  | curso_id |   | curso_id | profesor_nombre   |
+----------------+----------+   +----------+-------------------+
|              1 |      101 |   |      101 | Dr. López         |
|              2 |      101 |   |      102 | Dra. Martínez     |
|              1 |      102 |   +----------+-------------------+
+----------------+----------+
```

---

## 3. Tercera Forma Normal (3NF)

**Requisitos:**
1. ✅ Cumple 2NF
2. ✅ No hay **dependencias transitivas**

**Dependencia transitiva:**  
Cuando un atributo no-clave depende de otro atributo no-clave.

```
A → B → C

C depende de B, y B depende de A
Entonces C depende transitivamente de A
```

### Ejemplo

**❌ NO está en 3NF:**

```
Tabla: empleados
+----+-----------+-----------------+------------------+
| id | nombre    | departamento_id | departamento_jefe|
+----+-----------+-----------------+------------------+
|  1 | Ana       |             100 | Carlos           |
|  2 | Bob       |             100 | Carlos           |
|  3 | Eva       |             200 | Diana            |
+----+-----------+-----------------+------------------+
       ↑              ↑                    ↑
   PRIMARY KEY    no-clave          depende de departamento_id ← ❌
```

**Problema:**  
`departamento_jefe` depende de `departamento_id`, no de `id` (clave primaria).

**Dependencia transitiva:**
```
id → departamento_id → departamento_jefe
```

**✅ 3NF:**

```
Tabla: empleados                   Tabla: departamentos
+----+-----------+-----------------+  +-----------------+------------------+
| id | nombre    | departamento_id |  | departamento_id | departamento_jefe|
+----+-----------+-----------------+  +-----------------+------------------+
|  1 | Ana       |             100 |  |             100 | Carlos           |
|  2 | Bob       |             100 |  |             200 | Diana            |
|  3 | Eva       |             200 |  +-----------------+------------------+
+----+-----------+-----------------+
```

---

### Otro Ejemplo

**❌ NO está en 3NF:**

```
Tabla: productos
+----+-----------+-------------+----------------+
| id | nombre    | categoria   | categoria_desc |
+----+-----------+-------------+----------------+
|  1 | Laptop    | Electrónica | Equipos...     |
|  2 | Mouse     | Electrónica | Equipos...     |
|  3 | Mesa      | Muebles     | Mobiliario...  |
+----+-----------+-------------+----------------+
                      ↑               ↑
                  no-clave      depende de categoria ← ❌
```

**✅ 3NF:**

```
Tabla: productos                   Tabla: categorias
+----+-----------+-------------+   +-------------+----------------+
| id | nombre    | categoria   |   | nombre      | descripcion    |
+----+-----------+-------------+   +-------------+----------------+
|  1 | Laptop    | Electrónica |   | Electrónica | Equipos...     |
|  2 | Mouse     | Electrónica |   | Muebles     | Mobiliario...  |
|  3 | Mesa      | Muebles     |   +-------------+----------------+
+----+-----------+-------------+
```

---

## 4. Forma Normal de Boyce-Codd (BCNF)

**Requisitos:**
1. ✅ Cumple 3NF
2. ✅ Toda dependencia funcional debe tener como origen una superclave

**Nota:**  
BCNF es una versión más estricta de 3NF. Generalmente, llegar a 3NF es suficiente.

### Ejemplo

**❌ NO está en BCNF:**

```
Tabla: asignaciones_aula
+----------------+-------+-----------+
| estudiante_id  | aula  | profesor  |
+----------------+-------+-----------+
|              1 | A101  | Dr. López |
|              2 | A101  | Dr. López |
+----------------+-------+-----------+

Restricción: Un aula solo puede tener un profesor
Dependencia: aula → profesor
Problema: aula no es superclave
```

**✅ BCNF:**

```
Tabla: asignaciones              Tabla: aulas_profesor
+----------------+-------+       +-------+-----------+
| estudiante_id  | aula  |       | aula  | profesor  |
+----------------+-------+       +-------+-----------+
|              1 | A101  |       | A101  | Dr. López |
|              2 | A101  |       +-------+-----------+
+----------------+-------+
```

---

## Proceso Completo: Ejemplo

### Tabla original (sin normalizar)

```
Tabla: ventas
+----------+--------------+------------------+------------------+----------------+
| venta_id | producto     | cliente_nombre   | cliente_email    | precio         |
+----------+--------------+------------------+------------------+----------------+
|        1 | Laptop, Mouse| Ana García       | ana@example.com  | 1050           |
|        2 | Teclado      | Ana García       | ana@example.com  | 50             |
|        3 | Monitor      | Bob Smith        | bob@example.com  | 300            |
+----------+--------------+------------------+------------------+----------------+
```

**Problemas:**
- ❌ `producto` tiene múltiples valores (Laptop, Mouse)
- ❌ Datos de Ana duplicados

---

### Paso 1: Primera Forma Normal (1NF)

**Separar valores múltiples:**

```
Tabla: ventas
+----------+-----------+------------------+------------------+----------------+
| venta_id | producto  | cliente_nombre   | cliente_email    | precio         |
+----------+-----------+------------------+------------------+----------------+
|        1 | Laptop    | Ana García       | ana@example.com  | 1000           |
|        1 | Mouse     | Ana García       | ana@example.com  | 50             |
|        2 | Teclado   | Ana García       | ana@example.com  | 50             |
|        3 | Monitor   | Bob Smith        | bob@example.com  | 300            |
+----------+-----------+------------------+------------------+----------------+
```

✅ Ahora cada columna tiene un solo valor.

---

### Paso 2: Segunda Forma Normal (2NF)

**Eliminar dependencias parciales:**

```
Tabla: ventas                      Tabla: items_venta
+----------+------------------+    +----------+-----------+-------+
| venta_id | cliente_nombre   |    | venta_id | producto  | precio|
|          | cliente_email    |    +----------+-----------+-------+
+----------+------------------+    |        1 | Laptop    | 1000  |
|        1 | Ana García       |    |        1 | Mouse     |   50  |
|          | ana@example.com  |    |        2 | Teclado   |   50  |
|        2 | Ana García       |    |        3 | Monitor   |  300  |
|          | ana@example.com  |    +----------+-----------+-------+
|        3 | Bob Smith        |
|          | bob@example.com  |
+----------+------------------+
```

✅ Productos separados de ventas.

---

### Paso 3: Tercera Forma Normal (3NF)

**Eliminar dependencias transitivas:**

```
Tabla: ventas                      Tabla: items_venta
+----------+------------+           +----------+-----------+-------+
| venta_id | cliente_id |           | venta_id | producto  | precio|
+----------+------------+           +----------+-----------+-------+
|        1 |          1 |           |        1 | Laptop    | 1000  |
|        2 |          1 |           |        1 | Mouse     |   50  |
|        3 |          2 |           |        2 | Teclado   |   50  |
+----------+------------+           |        3 | Monitor   |  300  |
                                    +----------+-----------+-------+

Tabla: clientes
+------------+------------------+------------------+
| cliente_id | nombre           | email            |
+------------+------------------+------------------+
|          1 | Ana García       | ana@example.com  |
|          2 | Bob Smith        | bob@example.com  |
+------------+------------------+------------------+
```

✅ Cliente en tabla separada, sin duplicación.

---

## Ventajas de Normalización

### ✅ Elimina redundancia

```
-- Sin normalizar: nombre de Ana 3 veces
-- Normalizada: nombre de Ana 1 vez
```

### ✅ Evita anomalías

```
-- Actualizar email de Ana: solo 1 lugar
UPDATE clientes SET email = 'ana_nuevo@example.com'
WHERE cliente_id = 1;
```

### ✅ Integridad

```
-- Relaciones garantizadas por FOREIGN KEY
```

### ✅ Facilita mantenimiento

```
-- Cambiar estructura de clientes no afecta ventas
ALTER TABLE clientes ADD COLUMN telefono VARCHAR(20);
```

---

## Desventajas de Normalización

### ❌ Más JOINs

```sql
-- Sin normalizar: 1 tabla
SELECT * FROM ventas WHERE cliente_nombre = 'Ana García';

-- Normalizada: 2 tablas
SELECT v.*, c.nombre
FROM ventas v
JOIN clientes c ON v.cliente_id = c.cliente_id
WHERE c.nombre = 'Ana García';
```

### ❌ Puede ser más lento

```
Más tablas = más JOINs = más procesamiento
```

---

## Desnormalización

**¿Cuándo desnormalizar?**

A veces es mejor **no** normalizar para mejorar performance.

### Ejemplo: Contador de posts

**Normalizado:**

```sql
SELECT usuarios.nombre, COUNT(posts.id)
FROM usuarios
LEFT JOIN posts ON usuarios.id = posts.usuario_id
GROUP BY usuarios.nombre;
-- Query lento con millones de posts
```

**Desnormalizado:**

```sql
-- Agregar columna contador
ALTER TABLE usuarios ADD COLUMN posts_count INTEGER DEFAULT 0;

-- Actualizar contador al crear post
INSERT INTO posts (...) VALUES (...);
UPDATE usuarios SET posts_count = posts_count + 1 WHERE id = ?;

-- Query rápido
SELECT nombre, posts_count FROM usuarios;
```

**Trade-off:**
- ✅ Consultas mucho más rápidas
- ❌ Redundancia (contador duplicado)
- ❌ Riesgo de inconsistencia

---

## Reglas Prácticas

### Normaliza cuando:
- ✅ OLTP (Online Transaction Processing) - transacciones frecuentes
- ✅ Integridad es crítica (bancos, e-commerce)
- ✅ Datos cambian frecuentemente

### Desnormaliza cuando:
- ✅ OLAP (Online Analytical Processing) - reportes, analítica
- ✅ Performance es crítica
- ✅ Datos no cambian (históricos)

---

## Resumen del Proceso

```
Datos sin normalizar
    ↓
[1NF] Valores atómicos, sin grupos repetidos
    ↓
[2NF] Sin dependencias parciales
    ↓
[3NF] Sin dependencias transitivas
    ↓
[BCNF] Toda dependencia desde superclave
```

**Objetivo:**  
Llegar a **3NF** es generalmente suficiente.

---

## Ejercicio Práctico

### Normalizar esta tabla:

```
Tabla: pedidos
+----------+------------+--------------+--------------+----------------+
| pedido_id| fecha      | cliente_nombre| cliente_email| productos      |
+----------+------------+--------------+--------------+----------------+
|        1 | 2024-01-01 | Ana          | ana@e.com    | Laptop, Mouse  |
|        2 | 2024-01-02 | Ana          | ana@e.com    | Teclado        |
|        3 | 2024-01-03 | Bob          | bob@e.com    | Monitor        |
+----------+------------+--------------+--------------+----------------+
```

### Solución (3NF):

```sql
-- Tabla: clientes
CREATE TABLE clientes (
    cliente_id INTEGER PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(255) UNIQUE
);

-- Tabla: pedidos
CREATE TABLE pedidos (
    pedido_id INTEGER PRIMARY KEY,
    cliente_id INTEGER,
    fecha DATE,
    FOREIGN KEY (cliente_id) REFERENCES clientes(cliente_id)
);

-- Tabla: productos
CREATE TABLE productos (
    producto_id INTEGER PRIMARY KEY,
    nombre VARCHAR(200)
);

-- Tabla: items_pedido
CREATE TABLE items_pedido (
    pedido_id INTEGER,
    producto_id INTEGER,
    PRIMARY KEY (pedido_id, producto_id),
    FOREIGN KEY (pedido_id) REFERENCES pedidos(pedido_id),
    FOREIGN KEY (producto_id) REFERENCES productos(producto_id)
);
```

---

## Resumen

**Normalización:**
- ✅ Elimina redundancia
- ✅ Evita anomalías
- ✅ Garantiza integridad

**Formas Normales:**
- **1NF:** Valores atómicos, sin grupos repetidos
- **2NF:** Sin dependencias parciales
- **3NF:** Sin dependencias transitivas
- **BCNF:** Más estricta (opcional)

**Desnormalización:**
- ⚡ Mejora performance
- ⚠️ Introduce redundancia
- 🎯 Usar con cuidado

**Próximo módulo:** Diagramas ERD (visualizar estructura)

---

## Recursos

- **[Database Normalization](https://en.wikipedia.org/wiki/Database_normalization)** - Wikipedia
- **[Normal Forms](https://www.studytonight.com/dbms/database-normalization.php)** - Tutorial completo
- **[When to Denormalize](https://www.sisense.com/blog/database-denormalization/)** - Casos prácticos

Ahora sabes **cómo organizar datos correctamente**. 🎯

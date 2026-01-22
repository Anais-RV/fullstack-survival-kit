# Índices y Optimización en PostgreSQL

> **Acelerar queries con índices y técnicas de optimización**

---

## ¿Qué es un índice?

Un **índice** es una estructura de datos que acelera las búsquedas.

**Analogía:**  
Como el índice de un libro: en lugar de leer todo el libro, vas directo a la página.

**Sin índice (Full Table Scan):**
```
SELECT * FROM usuarios WHERE email = 'ana@email.com';
-- Escanea 1,000,000 filas → 500ms
```

**Con índice:**
```
CREATE INDEX idx_usuarios_email ON usuarios(email);

SELECT * FROM usuarios WHERE email = 'ana@email.com';
-- Usa índice → 5ms
```

---

## ¿Cuándo usar índices?

### ✅ Sí, crear índice

- Columnas en `WHERE` frecuentes
- Columnas en `JOIN` (foreign keys)
- Columnas en `ORDER BY`
- Columnas en `GROUP BY`
- Búsquedas de texto

### ❌ No crear índice

- Tablas pequeñas (< 1000 filas)
- Columnas rara vez consultadas
- Columnas con pocos valores distintos (ej: booleano)
- Columnas que cambian mucho (inserts/updates frecuentes)

**Regla de oro:** Índices aceleran **lecturas** pero ralentizan **escrituras**.

---

## Tipos de índices en PostgreSQL

### 1. B-tree (por defecto)

**Mejor para:**
- Comparaciones: `=`, `<`, `>`, `<=`, `>=`, `BETWEEN`
- Ordenamiento: `ORDER BY`
- Rangos

**Uso:** 95% de los casos

```sql
-- Índice simple
CREATE INDEX idx_usuarios_email ON usuarios(email);

-- Índice compuesto (múltiples columnas)
CREATE INDEX idx_pedidos_usuario_fecha ON pedidos(usuario_id, fecha);

-- Índice único
CREATE UNIQUE INDEX idx_usuarios_email_unique ON usuarios(email);
```

### 2. Hash

**Mejor para:**
- Solo igualdad: `=`

**No soporta:**
- Rangos, ordenamiento

```sql
CREATE INDEX idx_usuarios_email_hash ON usuarios USING HASH (email);
```

**Nota:** B-tree casi siempre es mejor que Hash en PostgreSQL.

### 3. GiST (Generalized Search Tree)

**Mejor para:**
- Tipos geométricos (point, box, polygon)
- Rangos (daterange, int4range)
- Full-text search

```sql
-- Índice para geometría
CREATE INDEX idx_lugares_ubicacion ON lugares USING GIST (ubicacion);

-- Índice para rangos
CREATE INDEX idx_reservas_periodo ON reservas USING GIST (periodo);
```

### 4. GIN (Generalized Inverted Index)

**Mejor para:**
- Arrays
- JSONB
- Full-text search
- Múltiples valores por fila

```sql
-- Índice para JSONB
CREATE INDEX idx_productos_metadata ON productos USING GIN (metadata);

-- Índice para arrays
CREATE INDEX idx_usuarios_emails ON usuarios USING GIN (emails);

-- Índice para full-text
CREATE INDEX idx_posts_contenido ON posts USING GIN (to_tsvector('spanish', contenido));
```

### 5. BRIN (Block Range INdex)

**Mejor para:**
- Tablas muy grandes (> 100 GB)
- Datos naturalmente ordenados (ej: timestamp)
- Índices muy pequeños

```sql
CREATE INDEX idx_logs_fecha ON logs USING BRIN (fecha);
```

---

## Crear índices

### Sintaxis básica

```sql
CREATE INDEX nombre_indice ON tabla (columna);
```

### Índice único

```sql
-- No permite duplicados
CREATE UNIQUE INDEX idx_usuarios_email ON usuarios(email);
```

### Índice compuesto

```sql
-- Múltiples columnas (orden importa!)
CREATE INDEX idx_pedidos_usuario_fecha ON pedidos(usuario_id, fecha);

-- Rápido para:
WHERE usuario_id = 1 AND fecha > '2024-01-01'  -- ✅
WHERE usuario_id = 1  -- ✅

-- Lento para:
WHERE fecha > '2024-01-01'  -- ❌ (primera columna no en WHERE)
```

### Índice parcial

```sql
-- Solo indexa filas que cumplen condición
CREATE INDEX idx_productos_activos ON productos(nombre) WHERE activo = true;

-- Usa menos espacio, más rápido para:
SELECT * FROM productos WHERE activo = true AND nombre = 'Laptop';
```

### Índice de expresión

```sql
-- Indexa resultado de expresión
CREATE INDEX idx_usuarios_email_lower ON usuarios(LOWER(email));

-- Ahora rápido:
SELECT * FROM usuarios WHERE LOWER(email) = 'ana@email.com';
```

### Índice descendente

```sql
-- Para ORDER BY DESC
CREATE INDEX idx_productos_precio_desc ON productos(precio DESC);
```

---

## Gestionar índices

### Listar índices

```sql
-- Índices de una tabla
\d usuarios

-- Todos los índices
SELECT 
    schemaname,
    tablename,
    indexname,
    indexdef
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY tablename, indexname;
```

### Tamaño de índices

```sql
-- Tamaño de un índice
SELECT pg_size_pretty(pg_relation_size('idx_usuarios_email'));

-- Índices más grandes
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC
LIMIT 10;
```

### Eliminar índice

```sql
DROP INDEX idx_usuarios_email;

-- Sin error si no existe
DROP INDEX IF EXISTS idx_usuarios_email;
```

### Reindexar

```sql
-- Reindexar un índice
REINDEX INDEX idx_usuarios_email;

-- Reindexar tabla completa
REINDEX TABLE usuarios;

-- Reindexar base de datos
REINDEX DATABASE nombre_bd;
```

---

## EXPLAIN - Analizar queries

**EXPLAIN** muestra el plan de ejecución de una query.

### EXPLAIN básico

```sql
EXPLAIN SELECT * FROM usuarios WHERE email = 'ana@email.com';
```

**Salida:**
```
Seq Scan on usuarios  (cost=0.00..18.50 rows=1 width=100)
  Filter: (email = 'ana@email.com'::text)
```

**Interpretación:**
- `Seq Scan` - Escaneo secuencial (sin índice) ❌
- `cost=0.00..18.50` - Costo estimado
- `rows=1` - Filas esperadas
- `width=100` - Bytes por fila

### EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE SELECT * FROM usuarios WHERE email = 'ana@email.com';
```

**Salida:**
```
Index Scan using idx_usuarios_email on usuarios  (cost=0.28..8.29 rows=1 width=100) (actual time=0.015..0.016 rows=1 loops=1)
  Index Cond: (email = 'ana@email.com'::text)
Planning Time: 0.082 ms
Execution Time: 0.035 ms
```

**Diferencia con EXPLAIN:**
- `actual time` - Tiempo real de ejecución
- `rows=1` - Filas reales (no estimadas)
- **Ejecución real:** EXPLAIN ANALYZE ejecuta la query

### Tipos de escaneo

**1. Seq Scan (Sequential Scan)**
```
Seq Scan on usuarios
```
- Escanea toda la tabla
- ❌ Lento para tablas grandes

**2. Index Scan**
```
Index Scan using idx_usuarios_email on usuarios
```
- Usa índice
- ✅ Rápido

**3. Index Only Scan**
```
Index Only Scan using idx_usuarios_email on usuarios
```
- Solo usa índice (no accede a tabla)
- ✅ Más rápido

**4. Bitmap Index Scan**
```
Bitmap Heap Scan on usuarios
  -> Bitmap Index Scan on idx_usuarios_edad
```
- Combina múltiples índices
- Para queries complejas

**5. Hash Join**
```
Hash Join
```
- Join con hash table
- Rápido para JOINs grandes

**6. Nested Loop**
```
Nested Loop
```
- Join con bucle anidado
- Rápido para pocos datos

---

## Optimizar queries

### 1. Usar índices

```sql
-- ❌ Lento: Sin índice
SELECT * FROM pedidos WHERE usuario_id = 123;

-- ✅ Rápido: Con índice
CREATE INDEX idx_pedidos_usuario ON pedidos(usuario_id);
SELECT * FROM pedidos WHERE usuario_id = 123;
```

### 2. Evitar funciones en WHERE

```sql
-- ❌ No usa índice
SELECT * FROM usuarios WHERE LOWER(email) = 'ana@email.com';

-- ✅ Usar índice de expresión
CREATE INDEX idx_usuarios_email_lower ON usuarios(LOWER(email));
SELECT * FROM usuarios WHERE LOWER(email) = 'ana@email.com';

-- O mejor: normalizar datos
UPDATE usuarios SET email = LOWER(email);
SELECT * FROM usuarios WHERE email = 'ana@email.com';
```

### 3. Usar LIMIT cuando sea posible

```sql
-- ❌ Trae todo
SELECT * FROM productos ORDER BY creado_en DESC;

-- ✅ Solo lo necesario
SELECT * FROM productos ORDER BY creado_en DESC LIMIT 10;
```

### 4. SELECT solo columnas necesarias

```sql
-- ❌ Trae todos los campos
SELECT * FROM usuarios;

-- ✅ Solo necesarios
SELECT id, nombre, email FROM usuarios;
```

### 5. Usar EXISTS en lugar de IN

```sql
-- ❌ Lento con muchos IDs
SELECT * FROM productos 
WHERE categoria_id IN (SELECT id FROM categorias WHERE activo = true);

-- ✅ Más rápido
SELECT * FROM productos p
WHERE EXISTS (
    SELECT 1 FROM categorias c 
    WHERE c.id = p.categoria_id AND c.activo = true
);
```

### 6. Índices compuestos en orden correcto

```sql
-- Query frecuente:
SELECT * FROM pedidos WHERE usuario_id = 1 AND fecha > '2024-01-01';

-- ✅ Índice en orden de selectividad
CREATE INDEX idx_pedidos_usuario_fecha ON pedidos(usuario_id, fecha);

-- ❌ Orden invertido es menos útil
CREATE INDEX idx_pedidos_fecha_usuario ON pedidos(fecha, usuario_id);
```

### 7. Usar UNION ALL en lugar de UNION

```sql
-- ❌ UNION elimina duplicados (más lento)
SELECT nombre FROM usuarios WHERE activo = true
UNION
SELECT nombre FROM clientes WHERE activo = true;

-- ✅ UNION ALL (más rápido si no hay duplicados)
SELECT nombre FROM usuarios WHERE activo = true
UNION ALL
SELECT nombre FROM clientes WHERE activo = true;
```

---

## VACUUM y ANALYZE

### VACUUM

**VACUUM** limpia espacio "muerto" de filas eliminadas o actualizadas.

```sql
-- Vacuum de una tabla
VACUUM usuarios;

-- Vacuum completo (más lento, bloquea tabla)
VACUUM FULL usuarios;

-- Vacuum de toda la BD
VACUUM;
```

**Por qué es necesario:**
- PostgreSQL no elimina filas físicamente en DELETE/UPDATE
- Marca filas como "muertas"
- VACUUM recupera ese espacio

### ANALYZE

**ANALYZE** actualiza estadísticas para el planificador de queries.

```sql
-- Analyze de una tabla
ANALYZE usuarios;

-- Analyze de toda la BD
ANALYZE;

-- Ambos juntos
VACUUM ANALYZE usuarios;
```

**Cuándo ejecutar:**
- Después de grandes cambios (bulk inserts/updates)
- Si EXPLAIN muestra estimaciones erróneas
- Periódicamente (autovacuum lo hace automáticamente)

### Autovacuum

PostgreSQL ejecuta VACUUM y ANALYZE automáticamente.

```sql
-- Ver configuración
SHOW autovacuum;

-- Ver última ejecución
SELECT 
    schemaname,
    relname,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables;
```

---

## Estadísticas de uso de índices

### Índices no usados

```sql
-- Índices que nunca se usan (candidatos a eliminar)
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS index_scans,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE 'pg_toast%'
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Tablas sin índices en FKs

```sql
-- Foreign keys sin índice (potencial problema de performance)
SELECT 
    tc.table_name,
    kcu.column_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu 
    ON tc.constraint_name = kcu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND NOT EXISTS (
      SELECT 1 
      FROM pg_indexes 
      WHERE tablename = tc.table_name 
        AND indexdef LIKE '%' || kcu.column_name || '%'
  );
```

### Índices más usados

```sql
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC
LIMIT 10;
```

---

## Caso práctico: E-commerce optimizado

### Estructura inicial (sin optimizar)

```sql
CREATE TABLE categorias (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL,
    descripcion TEXT,
    precio NUMERIC(10, 2) NOT NULL,
    stock INTEGER DEFAULT 0,
    categoria_id INTEGER REFERENCES categorias(id),
    activo BOOLEAN DEFAULT true,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    email VARCHAR(150) NOT NULL,
    nombre VARCHAR(100),
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    usuario_id INTEGER REFERENCES usuarios(id),
    total NUMERIC(10, 2),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE items_pedido (
    id SERIAL PRIMARY KEY,
    pedido_id INTEGER REFERENCES pedidos(id),
    producto_id INTEGER REFERENCES productos(id),
    cantidad INTEGER,
    precio NUMERIC(10, 2)
);

-- Insertar datos de prueba (100,000 productos, 10,000 usuarios, 50,000 pedidos)
```

### Queries lentas (antes de optimizar)

```sql
-- Query 1: Productos activos de una categoría
EXPLAIN ANALYZE
SELECT * FROM productos 
WHERE categoria_id = 5 AND activo = true;
-- Seq Scan on productos  (cost=0.00..2500.00 rows=100) (actual time=45.123..89.456 rows=100)
```

### Agregar índices

```sql
-- 1. Índice en foreign keys
CREATE INDEX idx_productos_categoria ON productos(categoria_id);
CREATE INDEX idx_pedidos_usuario ON pedidos(usuario_id);
CREATE INDEX idx_items_pedido ON items_pedido(pedido_id);
CREATE INDEX idx_items_producto ON items_pedido(producto_id);

-- 2. Índice para búsquedas frecuentes
CREATE INDEX idx_usuarios_email ON usuarios(email);

-- 3. Índice parcial para productos activos
CREATE INDEX idx_productos_activos_categoria ON productos(categoria_id) WHERE activo = true;

-- 4. Índice para ordenamiento por fecha
CREATE INDEX idx_pedidos_fecha ON pedidos(fecha DESC);

-- 5. Índice de texto para búsqueda
CREATE INDEX idx_productos_nombre ON productos USING GIN (to_tsvector('spanish', nombre));
```

### Queries optimizadas (después)

```sql
-- Query 1: Ahora usa índice
EXPLAIN ANALYZE
SELECT * FROM productos 
WHERE categoria_id = 5 AND activo = true;
-- Index Scan using idx_productos_activos_categoria on productos  (cost=0.29..12.45 rows=100) (actual time=0.015..0.234 rows=100)
-- ⚡ 89ms → 0.23ms (380x más rápido)

-- Query 2: JOINs optimizados
EXPLAIN ANALYZE
SELECT 
    p.id,
    p.fecha,
    u.nombre,
    u.email
FROM pedidos p
INNER JOIN usuarios u ON p.usuario_id = u.id
WHERE p.fecha > '2024-01-01'
ORDER BY p.fecha DESC
LIMIT 10;
-- Usa idx_pedidos_fecha e idx_pedidos_usuario
-- ⚡ 150ms → 2ms (75x más rápido)

-- Query 3: Búsqueda de texto
EXPLAIN ANALYZE
SELECT nombre, precio
FROM productos
WHERE to_tsvector('spanish', nombre) @@ to_tsquery('spanish', 'laptop');
-- Usa idx_productos_nombre
-- ⚡ 200ms → 5ms (40x más rápido)
```

### Resultado

| Query | Antes | Después | Mejora |
|---|---|---|---|
| Productos por categoría | 89ms | 0.23ms | 380x |
| Pedidos recientes | 150ms | 2ms | 75x |
| Búsqueda texto | 200ms | 5ms | 40x |

---

## Mejores prácticas

✅ **Indexar foreign keys**
```sql
CREATE INDEX idx_pedidos_usuario ON pedidos(usuario_id);
```

✅ **Índices compuestos en orden correcto**
```sql
-- Query: WHERE usuario_id = X AND fecha > Y
CREATE INDEX idx_pedidos_usuario_fecha ON pedidos(usuario_id, fecha);
```

✅ **Índices parciales para filtros comunes**
```sql
CREATE INDEX idx_productos_activos ON productos(nombre) WHERE activo = true;
```

✅ **VACUUM ANALYZE periódicamente**
```sql
VACUUM ANALYZE;
```

✅ **Monitorear índices no usados**
```sql
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
```

❌ **No crear índices en:**
- Tablas pequeñas (< 1000 filas)
- Columnas booleanas
- Columnas que cambian constantemente

❌ **No usar funciones en WHERE sin índice**
```sql
-- ❌ Lento
WHERE UPPER(nombre) = 'ANA'

-- ✅ Rápido
CREATE INDEX idx_nombre_upper ON tabla(UPPER(nombre));
```

---

## Resumen

**Índices:**
- B-tree (por defecto) - 95% de casos
- GIN - JSONB, arrays, full-text
- GiST - Geometría, rangos
- BRIN - Tablas enormes, datos ordenados

**Crear índices:**
- Foreign keys
- Columnas en WHERE, JOIN, ORDER BY
- Índices compuestos (orden importa)
- Índices parciales (menos espacio)

**Optimización:**
- EXPLAIN ANALYZE para analizar
- SELECT solo columnas necesarias
- LIMIT cuando sea posible
- VACUUM ANALYZE periódicamente

**Próximo módulo:** Transacciones y propiedades ACID

---

## Recursos

- **[Indexes](https://www.postgresql.org/docs/current/indexes.html)** - Documentación oficial
- **[EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)** - Análisis de queries
- **[VACUUM](https://www.postgresql.org/docs/current/routine-vacuuming.html)** - Mantenimiento

¡Tus queries ahora vuelan! ⚡

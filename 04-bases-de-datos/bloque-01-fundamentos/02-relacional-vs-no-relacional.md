# Relacional vs No Relacional

> **SQL vs NoSQL: ¿Cuándo usar cada una?**

---

## Introducción

Existen **dos grandes familias** de bases de datos:

```
Bases de Datos
├── Relacionales (SQL)
│   └── PostgreSQL, MySQL, SQLite
└── No Relacionales (NoSQL)
    ├── Documentos (MongoDB)
    ├── Clave-Valor (Redis)
    ├── Columnas (Cassandra)
    └── Grafos (Neo4j)
```

---

## Bases de Datos Relacionales (SQL)

### Características

**1. Datos organizados en tablas**

```
Tabla: usuarios
+----+--------+-------------------+------+
| id | nombre | email             | edad |
+----+--------+-------------------+------+
|  1 | Ana    | ana@example.com   |   25 |
|  2 | Bob    | bob@example.com   |   30 |
+----+--------+-------------------+------+

Tabla: posts
+----+------------+------------------+------------+
| id | usuario_id | titulo           | contenido  |
+----+------------+------------------+------------+
|  1 |          1 | Mi primer post   | Texto...   |
|  2 |          1 | Segundo post     | Más texto…|
|  3 |          2 | Post de Bob      | Hola...    |
+----+------------+------------------+------------+
```

**2. Relaciones entre tablas**

```
usuarios (1) ──── (N) posts
   ↓
un usuario tiene muchos posts
```

**3. Esquema fijo**

```sql
-- Debes definir estructura antes de insertar
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    edad INTEGER
);
```

**4. Usa SQL (Structured Query Language)**

```sql
-- Buscar posts de Ana
SELECT posts.* 
FROM posts 
JOIN usuarios ON posts.usuario_id = usuarios.id
WHERE usuarios.nombre = 'Ana';
```

---

### Ventajas de SQL

✅ **Consistencia garantizada (ACID)**
```python
# Transferencia bancaria: todo o nada
inicio_transaccion()
restar(cuenta_a, 100)
sumar(cuenta_b, 100)
confirmar()  # Si falla algo, se revierte TODO
```

✅ **Relaciones claras**
```sql
-- Fácil obtener datos relacionados
SELECT usuarios.nombre, COUNT(posts.id)
FROM usuarios
LEFT JOIN posts ON usuarios.id = posts.usuario_id
GROUP BY usuarios.nombre;
```

✅ **Sin duplicados**
```sql
-- Clave primaria garantiza unicidad
INSERT INTO usuarios (id, nombre) VALUES (1, 'Ana');
INSERT INTO usuarios (id, nombre) VALUES (1, 'Bob');  -- ❌ ERROR
```

✅ **Consultas complejas**
```sql
-- Agregaciones, filtros, ordenamiento
SELECT categoria, AVG(precio)
FROM productos
WHERE stock > 0
GROUP BY categoria
HAVING AVG(precio) > 100
ORDER BY AVG(precio) DESC;
```

---

### Desventajas de SQL

❌ **Esquema rígido**
```sql
-- Agregar columna requiere migración
ALTER TABLE usuarios ADD COLUMN telefono VARCHAR(20);
-- Puede ser lento con millones de registros
```

❌ **Escalado vertical (más difícil)**
```
Escalar = Comprar servidor más potente
(limitado por hardware físico)
```

❌ **Menos flexible para datos variables**
```sql
-- Todos los usuarios deben tener mismas columnas
-- No puedes tener campos opcionales fácilmente
```

---

## Bases de Datos No Relacionales (NoSQL)

### Características

**1. Sin esquema fijo**

```javascript
// Usuario 1
{
  "id": 1,
  "nombre": "Ana",
  "email": "ana@example.com",
  "hobbies": ["lectura", "yoga"]  // ← campo opcional
}

// Usuario 2
{
  "id": 2,
  "nombre": "Bob",
  "email": "bob@example.com",
  "edad": 30,
  "direccion": {  // ← estructura anidada
    "calle": "Main St",
    "ciudad": "Madrid"
  }
}
```

**2. Escalado horizontal (fácil)**

```
Servidor 1    Servidor 2    Servidor 3
    ↓             ↓             ↓
  Datos A      Datos B      Datos C
  
(distribuir datos entre múltiples servidores)
```

**3. Alta performance para operaciones simples**

```javascript
// Obtener usuario por ID (muy rápido)
db.usuarios.findOne({ _id: 1 })
```

**4. Flexible para cambios**

```javascript
// No necesitas migración
db.usuarios.insertOne({
  nombre: "Carlos",
  nuevoCampo: "valor"  // ← sin problema
})
```

---

### Tipos de NoSQL

## 1. Bases de datos de documentos

**Ejemplo:** MongoDB, CouchDB

**Estructura:**
```javascript
// Documento (similar a JSON)
{
  "_id": "user123",
  "nombre": "Ana García",
  "email": "ana@example.com",
  "posts": [
    {
      "titulo": "Mi primer post",
      "contenido": "Texto...",
      "fecha": "2024-01-01",
      "comentarios": [
        {"autor": "Bob", "texto": "Buen post"}
      ]
    }
  ]
}
```

**Características:**
- Cada documento puede tener estructura diferente
- Datos anidados (posts dentro de usuario)
- Sin joins (todo en un documento)

**Cuándo usar:**
- ✅ Catálogos de productos
- ✅ Perfiles de usuario
- ✅ CMS (Content Management System)
- ✅ Datos semi-estructurados

---

## 2. Bases de datos clave-valor

**Ejemplo:** Redis, DynamoDB, Memcached

**Estructura:**
```
clave              valor
----------------------------------
"usuario:1"    →  "{"nombre": "Ana", "edad": 25}"
"sesion:abc"   →  "{"user_id": 1, "expires": "..."}"
"contador:123" →  "42"
```

**Características:**
- Extremadamente rápido
- Simple: GET/SET
- Generalmente en memoria (RAM)

**Cuándo usar:**
- ✅ Caché (datos temporales)
- ✅ Sesiones de usuario
- ✅ Contadores en tiempo real
- ✅ Rate limiting

**Ejemplo práctico:**
```python
# Redis
redis.set("usuario:1", json.dumps({"nombre": "Ana"}))
usuario = json.loads(redis.get("usuario:1"))

# Caché
redis.setex("producto:100", 3600, precio)  # expira en 1 hora
```

---

## 3. Bases de datos de columnas

**Ejemplo:** Cassandra, HBase, Google BigTable

**Estructura:**
```
Row Key    | nombre | edad | email
-----------+--------+------+------------------
usuario:1  | Ana    | 25   | ana@example.com
usuario:2  | Bob    | 30   | bob@example.com
```

**Diferencia con SQL:**
- SQL lee filas completas
- Columnas lee solo columnas necesarias

**Cuándo usar:**
- ✅ Analítica de big data
- ✅ Logs masivos
- ✅ Time-series data
- ✅ Millones de escrituras por segundo

---

## 4. Bases de datos de grafos

**Ejemplo:** Neo4j, ArangoDB, Amazon Neptune

**Estructura:**
```
(Ana) ──amiga_de──> (Bob)
  │                    │
  └──sigue──> (Carlos) ┘
```

**Características:**
- Nodos (entidades)
- Aristas (relaciones)
- Consultas de relaciones rápidas

**Cuándo usar:**
- ✅ Redes sociales (amigos de amigos)
- ✅ Recomendaciones
- ✅ Detección de fraude
- ✅ Rutas y navegación

**Ejemplo:**
```cypher
// Neo4j (lenguaje Cypher)
// "Amigos de amigos que les gusta el mismo género musical"
MATCH (yo:Usuario {nombre: 'Ana'})
      -[:AMIGO*2]-(amigo)
      -[:LE_GUSTA]->(genero)<-[:LE_GUSTA]-(yo)
RETURN amigo.nombre
```

---

## Ventajas de NoSQL

✅ **Escalado horizontal**
```
Agregar más servidores = más capacidad
(más barato y flexible que SQL)
```

✅ **Esquema flexible**
```javascript
// Sin migraciones
db.usuarios.insertOne({
  nombre: "Ana",
  nuevoCampo: "valor"  // sin problema
})
```

✅ **Alta velocidad para operaciones simples**
```javascript
// Muy rápido
db.usuarios.findOne({ _id: "123" })
```

✅ **Maneja grandes volúmenes**
```
Millones de escrituras por segundo
(Cassandra, MongoDB)
```

---

## Desventajas de NoSQL

❌ **Sin garantías ACID fuertes**
```javascript
// Puede haber inconsistencias temporales
// (eventual consistency)
```

❌ **Relaciones complejas difíciles**
```javascript
// Equivalente a JOIN requiere múltiples queries
const usuario = db.usuarios.findOne({ _id: 1 })
const posts = db.posts.find({ usuario_id: usuario._id })
```

❌ **Duplicación de datos**
```javascript
// Para evitar joins, duplicas datos
{
  "post_id": 1,
  "titulo": "Mi post",
  "autor": {  // ← datos del autor duplicados
    "nombre": "Ana",
    "email": "ana@example.com"
  }
}
```

❌ **Menos herramientas y estándares**
```
SQL: estándar universal
NoSQL: cada BD tiene su propio lenguaje
```

---

## Comparación lado a lado

| Aspecto | SQL (Relacional) | NoSQL (No Relacional) |
|---------|------------------|------------------------|
| **Estructura** | Tablas con esquema fijo | Documentos, clave-valor, etc. |
| **Esquema** | Rígido (definir antes) | Flexible (cambiar fácil) |
| **Escalado** | Vertical (servidor grande) | Horizontal (muchos servidores) |
| **Relaciones** | Fáciles (JOIN) | Difíciles (manual) |
| **Consistencia** | ACID fuerte | Eventual (más débil) |
| **Consultas** | SQL estándar | Propio de cada BD |
| **Transacciones** | Sí, completas | Limitadas |
| **Velocidad** | Rápido para complejas | Muy rápido para simples |
| **Duplicación** | Minimizada | Común (por diseño) |

---

## ¿Cuándo usar SQL?

### Casos ideales para SQL

✅ **Datos estructurados y relaciones claras**
```
E-commerce: Productos → Pedidos → Clientes
            ↓
         Inventario
```

✅ **Transacciones complejas**
```
Transferencias bancarias, pagos, reservas
```

✅ **Consultas complejas**
```sql
SELECT categoria, AVG(precio)
FROM productos
WHERE stock > 0
GROUP BY categoria;
```

✅ **Integridad crítica**
```
Sistemas financieros, médicos, gubernamentales
```

✅ **Reportes y analítica**
```sql
-- Fácil generar reportes complejos
SELECT YEAR(fecha), SUM(monto)
FROM ventas
GROUP BY YEAR(fecha);
```

---

## ¿Cuándo usar NoSQL?

### Casos ideales para NoSQL

✅ **Datos semi-estructurados o variables**
```javascript
// Catálogo de productos con atributos diferentes
{
  "tipo": "laptop",
  "ram": "16GB",  // ← solo laptops
  "procesador": "Intel i7"
}
{
  "tipo": "libro",
  "autor": "García Márquez",  // ← solo libros
  "paginas": 300
}
```

✅ **Alta velocidad y volumen**
```
Logs, métricas, IoT, redes sociales
```

✅ **Escalabilidad horizontal**
```
Necesitas distribuir entre muchos servidores
```

✅ **Datos anidados**
```javascript
// Evitar múltiples tablas
{
  "usuario": "Ana",
  "posts": [
    {"titulo": "Post 1", "comentarios": [...]}
  ]
}
```

✅ **Prototipos rápidos**
```javascript
// Sin definir esquema antes
db.coleccion.insertOne({ cualquier: "cosa" })
```

---

## Ejemplos del mundo real

### SQL (PostgreSQL, MySQL)

| Aplicación | Motivo |
|------------|--------|
| **Bancos** | Transacciones ACID |
| **E-commerce** | Relaciones (productos, pedidos, clientes) |
| **ERP/CRM** | Datos estructurados y reportes |
| **WordPress** | Posts, usuarios, comentarios relacionados |

### NoSQL (MongoDB, Redis, etc.)

| Aplicación | Tipo | Motivo |
|------------|------|--------|
| **Netflix** | Documentos | Catálogos variables |
| **Facebook** | Grafos | Red social (amigos) |
| **Twitter** | Clave-Valor | Caché, sesiones |
| **Uber** | Documentos | Viajes en tiempo real |
| **Airbnb** | Documentos | Listados diversos |

---

## Híbridos: PostgreSQL + JSON

**PostgreSQL moderno:**

```sql
-- Tabla con columna JSON
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    metadata JSONB  -- ← columna flexible
);

-- Insertar con JSON
INSERT INTO usuarios (nombre, metadata)
VALUES ('Ana', '{"hobbies": ["yoga", "lectura"], "edad": 25}');

-- Consultar dentro del JSON
SELECT nombre
FROM usuarios
WHERE metadata->>'edad' = '25';
```

**Ventaja:** Lo mejor de ambos mundos
- ✅ ACID y relaciones (SQL)
- ✅ Flexibilidad (NoSQL)

---

## Casos de estudio

### Caso 1: E-commerce

**Opción A: Solo SQL**
```sql
productos → categorias
    ↓
pedidos → items_pedido
    ↓
clientes
```

**Pros:**
- ✅ Relaciones claras
- ✅ Integridad (stock, precios)
- ✅ Reportes fáciles

**Contras:**
- ❌ Muchos JOINs
- ❌ Lento con millones de productos

**Opción B: SQL + NoSQL**
```
SQL (PostgreSQL):
- Pedidos (crítico, transaccional)
- Clientes
- Inventario

NoSQL (MongoDB):
- Catálogo de productos (lectura rápida)
- Reseñas
- Sesiones de usuario (Redis)
```

**Ventaja:** Usa cada BD para su fortaleza

---

### Caso 2: Red social

**Opción A: Solo SQL**
```sql
usuarios → posts → comentarios → likes
    ↓
amistades (tabla intermedia)
```

**Pros:**
- ✅ Consistencia

**Contras:**
- ❌ "Amigos de amigos" requiere joins complejos
- ❌ Lento con millones de usuarios

**Opción B: SQL + Grafos**
```
SQL:
- Usuarios (datos básicos)
- Posts

NoSQL (Grafos - Neo4j):
- Relaciones sociales (amigos, seguidores)
- Recomendaciones
```

---

### Caso 3: Blog simple

**Mejor opción: SQL (PostgreSQL)**

```sql
usuarios → posts → comentarios
```

**Motivo:**
- Relaciones simples y claras
- No necesita escalar masivamente
- Consultas naturales con JOIN

**No necesitas NoSQL si:**
- ❌ Pocos usuarios (< 100,000)
- ❌ Estructura fija
- ❌ Sin necesidad de escalar horizontalmente

---

## Modelo CAP: Por qué no ambas

**Teorema CAP:** Solo puedes tener 2 de 3

```
    C (Consistency)
    /              \
   /                \
A (Availability)  P (Partition Tolerance)
```

**C - Consistency:** Todos ven los mismos datos  
**A - Availability:** Siempre responde  
**P - Partition Tolerance:** Funciona con fallos de red

### SQL: CP
```
Prioriza consistencia sobre disponibilidad
(puede no responder si no garantiza consistencia)
```

### NoSQL: AP
```
Prioriza disponibilidad sobre consistencia
(puede devolver datos desactualizados)
```

---

## Decisión práctica

### Usa SQL si:
- ✅ Relaciones complejas
- ✅ Transacciones críticas
- ✅ Datos estructurados
- ✅ Consultas complejas
- ✅ Equipos familiarizados con SQL

### Usa NoSQL si:
- ✅ Datos variables (esquema cambia)
- ✅ Alta velocidad y volumen
- ✅ Escalado horizontal necesario
- ✅ Datos anidados/jerárquicos
- ✅ Prototipo rápido

### Usa ambos si:
- ✅ Aplicación grande
- ✅ Diferentes necesidades por módulo
- ✅ Recursos para mantener dos sistemas

---

## Resumen

**SQL (Relacional):**
- 📊 Tablas con esquema fijo
- 🔗 Relaciones con JOIN
- ✅ ACID (consistencia fuerte)
- 🚀 Consultas complejas
- ⬆️ Escalado vertical

**NoSQL (No Relacional):**
- 📄 Documentos, clave-valor, grafos
- 🔀 Esquema flexible
- ⚡ Rápido para operaciones simples
- ↔️ Escalado horizontal
- 🌐 Eventual consistency

**Decisión:**
- La mayoría de aplicaciones: SQL (PostgreSQL)
- Big data, alta velocidad: NoSQL
- Aplicaciones grandes: Híbrido

**Próximo módulo:** Conceptos fundamentales (tablas, claves, relaciones)

---

## Recursos

- **[SQL vs NoSQL](https://www.mongodb.com/nosql-explained/nosql-vs-sql)** - Comparación oficial
- **[CAP Theorem](https://en.wikipedia.org/wiki/CAP_theorem)** - Teoría detrás
- **[When to use MongoDB](https://www.mongodb.com/when-to-use-mongodb)** - Casos de uso
- **[PostgreSQL JSON](https://www.postgresql.org/docs/current/datatype-json.html)** - Híbrido

Ahora sabes **cuándo usar SQL y cuándo NoSQL**. 🎯

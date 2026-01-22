# Introducción a Bases de Datos

> **¿Qué es una base de datos y por qué existen?**

---

## ¿Qué es una base de datos?

**Definición simple:**  
Una **base de datos** es una colección organizada de información que se puede acceder, gestionar y actualizar fácilmente.

**Analogía del mundo real:**
- Una **biblioteca** es una base de datos de libros
- Tu **agenda de contactos** es una base de datos de personas
- Un **archivo Excel** es una base de datos simple

---

## El problema que resuelven

### Antes de las bases de datos

**Opción 1: Archivos de texto**

```
usuarios.txt
------------
Ana García,ana@email.com,25
Bob Smith,bob@email.com,30
```

**Problemas:**
- ❌ Buscar es lento (leer todo el archivo)
- ❌ Actualizar es difícil (reescribir todo)
- ❌ No hay validaciones
- ❌ No hay relaciones entre datos
- ❌ Concurrencia (dos personas editando a la vez)

**Opción 2: Variables en memoria**

```python
usuarios = [
    {'nombre': 'Ana', 'email': 'ana@email.com'},
    {'nombre': 'Bob', 'email': 'bob@email.com'}
]
```

**Problemas:**
- ❌ Se pierden al cerrar el programa
- ❌ No escala (millones de registros)
- ❌ No compartible entre aplicaciones
- ❌ Sin respaldos automáticos

---

## ¿Por qué existen las bases de datos?

### 1. Persistencia
**Problema:** Los datos en memoria se pierden.  
**Solución:** BD guarda en disco permanentemente.

### 2. Organización
**Problema:** Archivos sin estructura.  
**Solución:** BD organiza en tablas relacionadas.

### 3. Búsqueda rápida
**Problema:** Buscar en archivo grande es lento.  
**Solución:** BD usa índices (como índice de libro).

### 4. Integridad
**Problema:** Datos inconsistentes o duplicados.  
**Solución:** BD valida y garantiza reglas.

### 5. Concurrencia
**Problema:** Múltiples usuarios editando a la vez.  
**Solución:** BD maneja acceso simultáneo.

### 6. Seguridad
**Problema:** Cualquiera puede leer/modificar archivos.  
**Solución:** BD controla permisos por usuario.

### 7. Respaldo y recuperación
**Problema:** Pérdida de datos por fallo.  
**Solución:** BD hace backups automáticos.

---

## Evolución histórica

### 1960s: Bases de datos jerárquicas
```
Empresa
├── Departamento 1
│   ├── Empleado A
│   └── Empleado B
└── Departamento 2
    └── Empleado C
```

**Problema:** Rígidas, difíciles de cambiar.

### 1970s: Modelo relacional (SQL)
**Inventor:** Edgar F. Codd (IBM)

```
Tabla: Empleados
+----+--------+----------------+
| id | nombre | departamento_id |
+----+--------+----------------+
|  1 | Ana    |              1 |
|  2 | Bob    |              1 |
+----+--------+----------------+

Tabla: Departamentos
+----+--------+
| id | nombre |
+----+--------+
|  1 | IT     |
|  2 | RRHH   |
+----+--------+
```

**Ventaja:** Flexibilidad, relaciones claras.

### 1980s-1990s: Era de SQL
- Oracle, MySQL, PostgreSQL, SQL Server
- SQL se vuelve estándar

### 2000s: NoSQL
**Motivo:** Big Data, escalabilidad web

Tipos:
- **Documentos:** MongoDB
- **Clave-valor:** Redis
- **Columnas:** Cassandra
- **Grafos:** Neo4j

### 2010s-Presente: Híbrido
- PostgreSQL con JSON
- Bases de datos distribuidas
- Cloud databases

---

## Componentes de un sistema de BD

```
┌─────────────────────────────────────┐
│        APLICACIÓN (Python)          │
│  (tu código, Django, Flask, etc.)   │
└──────────────┬──────────────────────┘
               │
               │ Queries (SQL)
               ▼
┌─────────────────────────────────────┐
│   SGBD (Sistema Gestor de BD)       │
│   (PostgreSQL, MySQL, SQLite)       │
│                                     │
│  ┌──────────────────────────────┐  │
│  │   Motor de consultas         │  │
│  │   (procesa SQL)              │  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │   Motor de almacenamiento    │  │
│  │   (lee/escribe disco)        │  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │   Gestor de transacciones    │  │
│  │   (garantiza consistencia)   │  │
│  └──────────────────────────────┘  │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│         DISCO DURO                  │
│   (archivos de datos físicos)       │
└─────────────────────────────────────┘
```

**SGBD:** Sistema Gestor de Bases de Datos  
**Ejemplos:** PostgreSQL, MySQL, MongoDB, Oracle

---

## Tipos de datos que almacenan

### 1. Datos estructurados
**Características:**
- Formato fijo (tablas)
- Esquema definido
- Fácil de consultar

**Ejemplo:**
```
Usuarios
+----+-----------+-------------------+------+
| id | nombre    | email             | edad |
+----+-----------+-------------------+------+
|  1 | Ana       | ana@example.com   |   25 |
|  2 | Bob       | bob@example.com   |   30 |
+----+-----------+-------------------+------+
```

### 2. Datos semi-estructurados
**Características:**
- Formato flexible (JSON, XML)
- Esquema variable
- Más flexibilidad

**Ejemplo:**
```json
{
  "id": 1,
  "nombre": "Ana",
  "email": "ana@example.com",
  "edad": 25,
  "hobbies": ["lectura", "yoga"]  ← Campo opcional
}
```

### 3. Datos no estructurados
**Características:**
- Sin formato (texto libre, imágenes)
- Difícil de consultar
- Requiere procesamiento

**Ejemplo:**
- Documentos de texto
- Imágenes, videos
- Logs sin formato

---

## Operaciones básicas (CRUD)

Toda base de datos permite:

```
C - CREATE   (Crear)
R - READ     (Leer)
U - UPDATE   (Actualizar)
D - DELETE   (Eliminar)
```

**Ejemplo:**

```python
# CREATE
crear_usuario(nombre="Ana", email="ana@example.com")

# READ
usuario = obtener_usuario(id=1)

# UPDATE
actualizar_usuario(id=1, edad=26)

# DELETE
eliminar_usuario(id=1)
```

---

## Propiedades ACID

Bases de datos relacionales garantizan **ACID**:

### A - Atomicity (Atomicidad)
**Todo o nada**

```python
# Transferencia bancaria
inicio_transaccion()
restar_dinero(cuenta_origen, 100)
sumar_dinero(cuenta_destino, 100)
confirmar_transaccion()

# Si falla algo, TODO se revierte
```

### C - Consistency (Consistencia)
**Reglas siempre se cumplen**

```python
# Regla: saldo no puede ser negativo
# La BD no permitirá operaciones que rompan esta regla
```

### I - Isolation (Aislamiento)
**Transacciones no se interfieren**

```python
# Usuario A y Usuario B modifican datos
# No se pisan entre ellos
```

### D - Durability (Durabilidad)
**Datos confirmados persisten**

```python
# Después de confirmar_transaccion()
# Datos sobreviven a caídas del sistema
```

---

## Casos de uso

### E-commerce
```
Productos → Pedidos → Clientes
       ↓
   Inventario
```

### Red Social
```
Usuarios → Posts → Comentarios → Likes
    ↓
  Amigos (relaciones)
```

### Banco
```
Cuentas → Transacciones → Clientes
    ↓
  Movimientos
```

### Streaming (Netflix)
```
Usuarios → Perfiles → Visualizaciones
              ↓
          Películas/Series
```

---

## Comparación: Archivo vs Base de Datos

| Aspecto | Archivo | Base de Datos |
|---------|---------|---------------|
| **Persistencia** | ✅ Sí | ✅ Sí |
| **Búsqueda** | 🐌 Lenta | ⚡ Rápida |
| **Organización** | ❌ Manual | ✅ Automática |
| **Integridad** | ❌ No | ✅ Garantizada |
| **Concurrencia** | ❌ Limitada | ✅ Múltiples usuarios |
| **Seguridad** | ⚠️ Básica | ✅ Avanzada |
| **Respaldo** | ❌ Manual | ✅ Automático |
| **Escalabilidad** | ❌ Limitada | ✅ Millones de registros |
| **Relaciones** | ❌ Complejas | ✅ Naturales |

---

## Cuándo usar cada solución

### Usar archivos cuando:
- ✅ Datos muy simples
- ✅ Sin concurrencia
- ✅ Prototipo rápido
- ✅ Configuraciones

**Ejemplo:** Archivo de configuración `.env`

### Usar base de datos cuando:
- ✅ Datos complejos
- ✅ Múltiples usuarios
- ✅ Necesitas búsquedas rápidas
- ✅ Integridad es crítica
- ✅ Producción

**Ejemplo:** Casi cualquier aplicación web

---

## Ejemplo práctico: Blog

### Sin base de datos (archivo)

```python
# posts.txt
1|Mi primer post|Este es el contenido...|2024-01-01
2|Segundo post|Más contenido...|2024-01-02
```

**Problemas:**
- ¿Cómo buscar posts de un autor?
- ¿Cómo relacionar posts con comentarios?
- ¿Cómo garantizar que el ID sea único?

### Con base de datos

```sql
-- Tabla: posts
+----+------------------+-----------------------+------------+
| id | titulo           | contenido             | fecha      |
+----+------------------+-----------------------+------------+
|  1 | Mi primer post   | Este es el contenido…|2024-01-01 |
|  2 | Segundo post     | Más contenido…       |2024-01-02 |
+----+------------------+-----------------------+------------+

-- Búsqueda rápida:
SELECT * FROM posts WHERE fecha > '2024-01-01';

-- ID automático:
INSERT INTO posts (titulo, contenido) VALUES ('Nuevo', 'Texto');
-- ID se genera automáticamente
```

---

## Vocabulario básico

| Término | Significado |
|---------|-------------|
| **Base de datos** | Colección organizada de datos |
| **SGBD** | Software que gestiona la BD (PostgreSQL, MySQL) |
| **Tabla** | Estructura que almacena datos (como Excel) |
| **Fila/Registro** | Un elemento de datos |
| **Columna/Campo** | Un atributo de los datos |
| **Clave primaria** | Identificador único de registro |
| **SQL** | Lenguaje para hablar con la BD |
| **Query** | Consulta a la BD |
| **Esquema** | Estructura de la BD (qué tablas, columnas) |

---

## Analogía final: Biblioteca

```
Base de Datos = Biblioteca
├── Tablas = Secciones (Ficción, No-ficción)
├── Filas = Libros individuales
├── Columnas = Atributos (Título, Autor, ISBN)
├── Clave primaria = ISBN (único por libro)
├── Índice = Catálogo (buscar rápido)
└── Query = "Dame todos los libros de ciencia ficción"
```

---

## Resumen

**Una base de datos es:**
- ✅ Almacenamiento persistente y organizado
- ✅ Búsquedas rápidas con índices
- ✅ Garantiza integridad y consistencia
- ✅ Maneja concurrencia
- ✅ Escalable a millones de registros

**Existe para resolver:**
- ❌ Lentitud de archivos
- ❌ Pérdida de datos
- ❌ Datos inconsistentes
- ❌ Dificultad de relaciones

**Próximo módulo:** Relacionales vs No Relacionales

---

## Recursos

- **[SQL vs NoSQL](https://www.youtube.com/watch?v=Q_9cX9aJQn0)** - Video explicativo
- **[ACID Properties](https://en.wikipedia.org/wiki/ACID)** - Wikipedia
- **[Database Design](https://www.lucidchart.com/pages/database-diagram/database-design)** - Guía visual

Ahora entiendes **por qué** existen las bases de datos. 💾

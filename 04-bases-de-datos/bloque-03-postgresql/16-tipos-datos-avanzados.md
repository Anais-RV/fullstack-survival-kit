# Tipos de Datos Avanzados en PostgreSQL

> **JSON, Arrays, UUID, ENUM y tipos especiales de PostgreSQL**

---

## Diferencias con SQLite

**SQLite** tiene tipos limitados: INTEGER, TEXT, REAL, BLOB

**PostgreSQL** tiene 40+ tipos nativos:
- ✅ JSON/JSONB
- ✅ Arrays
- ✅ UUID
- ✅ ENUM
- ✅ Rangos (daterange, int4range)
- ✅ Geométricos (point, line, polygon)
- ✅ Network (inet, macaddr)
- ✅ Y más...

---

## JSON y JSONB

**JSON** almacena texto JSON tal cual.  
**JSONB** almacena JSON en formato binario (más eficiente).

**Recomendación:** Usar **JSONB** siempre.

### Crear tabla con JSONB

```sql
CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(200),
    precio NUMERIC(10, 2),
    metadata JSONB
);
```

### Insertar datos JSON

```sql
INSERT INTO productos (nombre, precio, metadata) VALUES
    ('Laptop', 1200, '{"marca": "Dell", "ram": 16, "ssd": 512, "colores": ["negro", "plata"]}'),
    ('Mouse', 25, '{"marca": "Logitech", "inalambrico": true, "dpi": 1600}'),
    ('Teclado', 80, '{"marca": "Corsair", "mecanico": true, "switches": "Cherry MX Blue"}');
```

### Consultar JSON

```sql
-- Obtener campo (devuelve JSON)
SELECT metadata -> 'marca' FROM productos;
-- Resultado: "Dell", "Logitech", "Corsair" (con comillas)

-- Obtener campo como texto
SELECT metadata ->> 'marca' FROM productos;
-- Resultado: Dell, Logitech, Corsair (sin comillas)

-- Navegar objetos anidados
SELECT metadata -> 'specs' -> 'cpu' FROM productos;

-- Obtener elemento de array
SELECT metadata -> 'colores' -> 0 FROM productos;
-- Resultado: "negro"

-- Extraer como texto
SELECT metadata -> 'colores' ->> 0 FROM productos;
-- Resultado: negro
```

### Filtrar por JSON

```sql
-- Productos de marca Dell
SELECT nombre, precio
FROM productos
WHERE metadata ->> 'marca' = 'Dell';

-- Productos con RAM >= 16
SELECT nombre, metadata ->> 'ram' AS ram
FROM productos
WHERE (metadata ->> 'ram')::INTEGER >= 16;

-- Verificar si existe clave
SELECT nombre
FROM productos
WHERE metadata ? 'inalambrico';

-- Verificar múltiples claves
SELECT nombre
FROM productos
WHERE metadata ?| array['inalambrico', 'mecanico'];  -- O (alguna)

SELECT nombre
FROM productos
WHERE metadata ?& array['marca', 'ram'];  -- Y (todas)
```

### Operadores JSONB

| Operador | Descripción | Ejemplo |
|---|---|---|
| `->` | Obtener valor JSON | `metadata -> 'marca'` |
| `->>` | Obtener texto | `metadata ->> 'marca'` |
| `#>` | Path JSON | `metadata #> '{specs,cpu}'` |
| `#>>` | Path texto | `metadata #>> '{specs,cpu}'` |
| `?` | Existe clave | `metadata ? 'marca'` |
| `?|` | Existe alguna clave | `metadata ?| array['a','b']` |
| `?&` | Existen todas las claves | `metadata ?& array['a','b']` |
| `@>` | Contiene JSON | `metadata @> '{"marca":"Dell"}'` |
| `<@` | Contenido en JSON | `'{"marca":"Dell"}' <@ metadata` |

### Funciones JSONB

```sql
-- Claves de objeto
SELECT jsonb_object_keys(metadata) FROM productos;

-- Valores de objeto
SELECT jsonb_each(metadata) FROM productos;

-- Longitud de array
SELECT jsonb_array_length(metadata -> 'colores') FROM productos;

-- Construir JSON
SELECT jsonb_build_object(
    'nombre', nombre,
    'precio', precio,
    'marca', metadata ->> 'marca'
) FROM productos;

-- Agregar/modificar campo
UPDATE productos
SET metadata = metadata || '{"descuento": 10}'
WHERE id = 1;

-- Eliminar campo
UPDATE productos
SET metadata = metadata - 'descuento'
WHERE id = 1;

-- Reemplazar campo anidado
UPDATE productos
SET metadata = jsonb_set(metadata, '{specs,ram}', '32')
WHERE id = 1;
```

### Índices en JSONB

```sql
-- Índice GIN (recomendado)
CREATE INDEX idx_productos_metadata ON productos USING GIN (metadata);

-- Ahora queries con @>, ?, ?|, ?& son rápidas
SELECT * FROM productos WHERE metadata @> '{"marca": "Dell"}';
```

---

## Arrays

PostgreSQL soporta **arrays multidimensionales** de cualquier tipo.

### Crear tabla con arrays

```sql
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    emails TEXT[],
    telefonos VARCHAR(20)[],
    numeros_favoritos INTEGER[]
);
```

### Insertar arrays

```sql
-- Sintaxis con llaves
INSERT INTO usuarios (nombre, emails, telefonos, numeros_favoritos) VALUES
    ('Ana', 
     ARRAY['ana@email.com', 'ana@work.com'], 
     ARRAY['555-1234', '555-5678'],
     ARRAY[7, 13, 42]);

-- Sintaxis alternativa
INSERT INTO usuarios (nombre, emails) VALUES
    ('Bob', '{"bob@email.com", "bob@personal.com"}');

-- Array vacío
INSERT INTO usuarios (nombre, emails) VALUES
    ('Carlos', '{}');
```

### Consultar arrays

```sql
-- Primer elemento (índice 1, no 0!)
SELECT nombre, emails[1] FROM usuarios;

-- Segundo teléfono
SELECT nombre, telefonos[2] FROM usuarios;

-- Slice (subarray)
SELECT nombre, emails[1:2] FROM usuarios;

-- Longitud de array
SELECT nombre, array_length(emails, 1) AS num_emails FROM usuarios;

-- Descomponer array en filas
SELECT nombre, unnest(emails) AS email FROM usuarios;
```

### Filtrar por arrays

```sql
-- Usuarios con email específico
SELECT nombre
FROM usuarios
WHERE 'ana@email.com' = ANY(emails);

-- Usuarios con todos estos emails
SELECT nombre
FROM usuarios
WHERE emails @> ARRAY['ana@email.com', 'ana@work.com'];

-- Usuarios con al menos uno de estos emails
SELECT nombre
FROM usuarios
WHERE emails && ARRAY['bob@email.com', 'test@email.com'];

-- Usuarios con exactamente 2 emails
SELECT nombre
FROM usuarios
WHERE array_length(emails, 1) = 2;
```

### Modificar arrays

```sql
-- Agregar elemento
UPDATE usuarios
SET emails = array_append(emails, 'nuevo@email.com')
WHERE nombre = 'Ana';

-- Agregar al inicio
UPDATE usuarios
SET emails = array_prepend('primero@email.com', emails)
WHERE nombre = 'Ana';

-- Concatenar arrays
UPDATE usuarios
SET emails = emails || ARRAY['otro@email.com']
WHERE nombre = 'Ana';

-- Eliminar elemento
UPDATE usuarios
SET emails = array_remove(emails, 'ana@work.com')
WHERE nombre = 'Ana';

-- Reemplazar elemento
UPDATE usuarios
SET emails[1] = 'nueva_direccion@email.com'
WHERE nombre = 'Ana';
```

### Índices en arrays

```sql
-- Índice GIN para búsquedas
CREATE INDEX idx_usuarios_emails ON usuarios USING GIN (emails);

-- Ahora búsquedas con ANY, @>, && son rápidas
SELECT * FROM usuarios WHERE 'test@email.com' = ANY(emails);
```

---

## UUID (Universally Unique Identifier)

**UUID** son identificadores únicos de 128 bits.

**Ventajas:**
- ✅ Únicos globalmente (no solo en tu BD)
- ✅ No exponen cantidad de registros
- ✅ Seguros para APIs públicas
- ✅ Distribuibles (múltiples servidores)

### Habilitar extensión

```sql
-- Habilitar generación de UUIDs
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- O con pgcrypto (más moderno)
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

### Crear tabla con UUID

```sql
CREATE TABLE articulos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    titulo VARCHAR(200),
    contenido TEXT,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Insertar con UUID

```sql
-- UUID automático
INSERT INTO articulos (titulo, contenido) VALUES
    ('Mi primer artículo', 'Contenido del artículo...');

-- UUID manual
INSERT INTO articulos (id, titulo, contenido) VALUES
    ('a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11', 'Segundo artículo', 'Más contenido...');
```

### Consultar

```sql
-- Por UUID
SELECT * FROM articulos 
WHERE id = 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11';

-- Generar UUID manualmente
SELECT gen_random_uuid();
```

### UUID vs SERIAL

| Característica | SERIAL (INTEGER) | UUID |
|---|---|---|
| **Tamaño** | 4 bytes | 16 bytes |
| **Predecible** | Sí (1, 2, 3...) | No |
| **Seguridad** | Baja | Alta |
| **Distribuible** | No | Sí |
| **Performance** | Más rápido | Ligeramente más lento |
| **Índice** | Más eficiente | Más espacio |

---

## ENUM (Enumeraciones)

**ENUM** es un tipo personalizado con valores predefinidos.

### Crear tipo ENUM

```sql
-- Crear tipo
CREATE TYPE estado_pedido AS ENUM ('pendiente', 'pagado', 'enviado', 'entregado', 'cancelado');

-- Usar en tabla
CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER,
    total NUMERIC(10, 2),
    estado estado_pedido DEFAULT 'pendiente',
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Insertar con ENUM

```sql
-- Valor válido
INSERT INTO pedidos (cliente_id, total, estado) VALUES
    (1, 150.00, 'pagado');

-- Error: valor inválido
INSERT INTO pedidos (cliente_id, total, estado) VALUES
    (2, 200.00, 'procesando');  -- ❌ ERROR
```

### Consultar ENUM

```sql
-- Filtrar por estado
SELECT * FROM pedidos WHERE estado = 'pendiente';

-- Todos los valores posibles
SELECT unnest(enum_range(NULL::estado_pedido));

-- Comparar (orden de definición)
SELECT * FROM pedidos WHERE estado > 'pagado';
-- Devuelve: enviado, entregado (posteriores en definición)
```

### Modificar ENUM

```sql
-- Agregar valor al final
ALTER TYPE estado_pedido ADD VALUE 'reembolsado';

-- Agregar antes de otro valor
ALTER TYPE estado_pedido ADD VALUE 'preparando' BEFORE 'enviado';

-- ❌ No se puede eliminar valores
-- ❌ No se puede renombrar valores
-- Solución: Crear nuevo tipo y migrar
```

### ENUM vs VARCHAR con CHECK

**ENUM:**
- ✅ Más eficiente (almacena como entero internamente)
- ✅ Validación automática
- ❌ Difícil de modificar

**VARCHAR + CHECK:**
- ✅ Fácil de modificar
- ❌ Menos eficiente
- ✅ Más flexible

```sql
-- Alternativa con CHECK
CREATE TABLE pedidos_alt (
    id SERIAL PRIMARY KEY,
    estado VARCHAR(20) CHECK (estado IN ('pendiente', 'pagado', 'enviado', 'entregado', 'cancelado'))
);
```

---

## Rangos (Range Types)

**Rangos** representan intervalos de valores.

### Tipos de rangos nativos

```sql
-- int4range - Rango de enteros
-- int8range - Rango de enteros largos
-- numrange - Rango de numéricos
-- tsrange - Rango de timestamps (sin timezone)
-- tstzrange - Rango de timestamps (con timezone)
-- daterange - Rango de fechas
```

### Crear tabla con rangos

```sql
CREATE TABLE reservas (
    id SERIAL PRIMARY KEY,
    habitacion_id INTEGER,
    cliente VARCHAR(100),
    periodo daterange,
    EXCLUDE USING GIST (habitacion_id WITH =, periodo WITH &&)  -- No solapamiento
);
```

### Insertar rangos

```sql
-- Inclusivo-Exclusivo [inicio, fin)
INSERT INTO reservas (habitacion_id, cliente, periodo) VALUES
    (101, 'Ana García', '[2024-01-15, 2024-01-20)'),
    (102, 'Bob Smith', '[2024-01-18, 2024-01-25)');

-- Intentar solapar (error por EXCLUDE constraint)
INSERT INTO reservas (habitacion_id, cliente, periodo) VALUES
    (101, 'Carlos', '[2024-01-17, 2024-01-22)');  -- ❌ ERROR
```

### Consultar rangos

```sql
-- Reservas activas en una fecha
SELECT cliente, periodo
FROM reservas
WHERE periodo @> '2024-01-18'::DATE;

-- Reservas que se solapan con un periodo
SELECT cliente, periodo
FROM reservas
WHERE periodo && '[2024-01-16, 2024-01-21)'::daterange;

-- Reservas completamente contenidas en un periodo
SELECT cliente, periodo
FROM reservas
WHERE periodo <@ '[2024-01-01, 2024-02-01)'::daterange;
```

### Operadores de rangos

| Operador | Descripción | Ejemplo |
|---|---|---|
| `@>` | Contiene elemento/rango | `'[1,5)' @> 3` → true |
| `<@` | Contenido en | `3 <@ '[1,5)'` → true |
| `&&` | Se solapan | `'[1,5)' && '[3,7)'` → true |
| `<<` | Estrictamente a la izquierda | `'[1,3)' << '[5,7)'` → true |
| `>>` | Estrictamente a la derecha | `'[5,7)' >> '[1,3)'` → true |
| `&<` | No se extiende a la derecha | `'[1,5)' &< '[3,7)'` → true |
| `&>` | No se extiende a la izquierda | `'[3,7)' &> '[1,5)'` → true |
| `-|-` | Adyacentes | `'[1,3)' -|- '[3,5)'` → true |
| `+` | Unión | `'[1,3)' + '[3,5)'` → `[1,5)` |
| `*` | Intersección | `'[1,5)' * '[3,7)'` → `[3,5)` |
| `-` | Diferencia | `'[1,5)' - '[3,7)'` → `[1,3)` |

### Funciones de rangos

```sql
-- Límite inferior
SELECT lower(periodo) FROM reservas;

-- Límite superior
SELECT upper(periodo) FROM reservas;

-- Está vacío
SELECT isempty('[1,1)'::int4range);  -- true

-- Duración (solo daterange, tsrange)
SELECT upper(periodo) - lower(periodo) AS duracion FROM reservas;
```

---

## Tipos geométricos

PostgreSQL tiene tipos para geometría 2D.

### Tipos geométricos

```sql
-- point - Punto (x, y)
-- line - Línea infinita
-- lseg - Segmento de línea
-- box - Rectángulo
-- path - Ruta (abierta o cerrada)
-- polygon - Polígono
-- circle - Círculo
```

### Ejemplo: Puntos

```sql
CREATE TABLE lugares (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    ubicacion POINT
);

-- Insertar puntos
INSERT INTO lugares (nombre, ubicacion) VALUES
    ('Casa', POINT(10, 20)),
    ('Oficina', POINT(15, 25)),
    ('Parque', POINT(5, 30));

-- Calcular distancia
SELECT 
    nombre,
    ubicacion <-> POINT(10, 20) AS distancia
FROM lugares
ORDER BY distancia;

-- Lugares dentro de un círculo
SELECT nombre
FROM lugares
WHERE ubicacion <@ CIRCLE(POINT(10, 20), 10);  -- Centro (10,20), radio 10
```

---

## Otros tipos especiales

### INET (Direcciones IP)

```sql
CREATE TABLE conexiones (
    id SERIAL PRIMARY KEY,
    usuario VARCHAR(100),
    ip_address INET,
    conectado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO conexiones (usuario, ip_address) VALUES
    ('ana', '192.168.1.100'),
    ('bob', '2001:db8::1');

-- Filtrar por subred
SELECT usuario, ip_address
FROM conexiones
WHERE ip_address << '192.168.1.0/24';
```

### MACADDR (Direcciones MAC)

```sql
CREATE TABLE dispositivos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    mac_address MACADDR
);

INSERT INTO dispositivos (nombre, mac_address) VALUES
    ('Laptop Ana', '08:00:2b:01:02:03'),
    ('Laptop Bob', '08-00-2b-01-02-04');
```

### MONEY (Moneda)

```sql
CREATE TABLE facturas (
    id SERIAL PRIMARY KEY,
    monto MONEY
);

INSERT INTO facturas (monto) VALUES
    (150.50),
    ('$1,250.99');

-- Formateo automático según locale
SELECT monto FROM facturas;
-- Resultado: $150.50, $1,250.99
```

---

## Tipos compuestos personalizados

```sql
-- Crear tipo compuesto
CREATE TYPE direccion AS (
    calle VARCHAR(200),
    ciudad VARCHAR(100),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50)
);

-- Usar en tabla
CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    direccion_envio direccion
);

-- Insertar
INSERT INTO clientes (nombre, direccion_envio) VALUES
    ('Ana García', ROW('Calle Mayor 123', 'Madrid', '28001', 'España'));

-- Consultar campos
SELECT nombre, (direccion_envio).ciudad FROM clientes;

-- Filtrar
SELECT nombre
FROM clientes
WHERE (direccion_envio).pais = 'España';
```

---

## Caso práctico: E-commerce completo

```sql
-- Tipos personalizados
CREATE TYPE estado_pedido AS ENUM ('pendiente', 'pagado', 'enviado', 'entregado', 'cancelado');

-- Productos con JSON
CREATE TABLE productos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre VARCHAR(200) NOT NULL,
    precio NUMERIC(10, 2) NOT NULL,
    tags TEXT[],
    especificaciones JSONB,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Usuarios con arrays
CREATE TABLE usuarios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre VARCHAR(100),
    emails TEXT[],
    telefono VARCHAR(20),
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Pedidos con ENUM
CREATE TABLE pedidos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID REFERENCES usuarios(id),
    total NUMERIC(10, 2),
    estado estado_pedido DEFAULT 'pendiente',
    metadata JSONB,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Índices
CREATE INDEX idx_productos_tags ON productos USING GIN (tags);
CREATE INDEX idx_productos_specs ON productos USING GIN (especificaciones);
CREATE INDEX idx_pedidos_estado ON pedidos (estado);

-- Insertar datos
INSERT INTO productos (nombre, precio, tags, especificaciones) VALUES
    ('Laptop Dell XPS 15',
     1299.99,
     ARRAY['electrónica', 'computadoras', 'dell'],
     '{"marca": "Dell", "modelo": "XPS 15", "ram": 16, "ssd": 512, "pantalla": "15.6 pulgadas", "procesador": "Intel i7"}'),
    
    ('Mouse Logitech MX Master',
     79.99,
     ARRAY['accesorios', 'periféricos', 'logitech'],
     '{"marca": "Logitech", "tipo": "inalámbrico", "dpi": 4000, "botones": 7, "bateria": "recargable"}');

INSERT INTO usuarios (nombre, emails, telefono) VALUES
    ('Ana García', 
     ARRAY['ana@email.com', 'ana.garcia@work.com'],
     '+34 600 123 456');

INSERT INTO pedidos (usuario_id, total, estado, metadata)
SELECT 
    id,
    1379.98,
    'pagado',
    '{"metodo_pago": "tarjeta", "envio_express": true, "direccion": "Calle Mayor 123, Madrid"}'
FROM usuarios WHERE nombre = 'Ana García';

-- Queries
-- 1. Productos de marca Dell
SELECT nombre, precio, especificaciones ->> 'marca' AS marca
FROM productos
WHERE especificaciones @> '{"marca": "Dell"}';

-- 2. Productos con tag "electrónica"
SELECT nombre, tags
FROM productos
WHERE 'electrónica' = ANY(tags);

-- 3. Productos con RAM >= 16
SELECT nombre, especificaciones ->> 'ram' AS ram
FROM productos
WHERE (especificaciones ->> 'ram')::INTEGER >= 16;

-- 4. Pedidos pagados con envío express
SELECT 
    u.nombre,
    p.total,
    p.metadata ->> 'envio_express' AS express
FROM pedidos p
INNER JOIN usuarios u ON p.usuario_id = u.id
WHERE p.estado = 'pagado'
  AND p.metadata ->> 'envio_express' = 'true';

-- 5. Usuarios con múltiples emails
SELECT nombre, array_length(emails, 1) AS num_emails
FROM usuarios
WHERE array_length(emails, 1) > 1;
```

---

## Resumen

**JSON/JSONB:**
- JSONB para datos semi-estructurados
- Operadores: `->`, `->>`, `@>`, `?`
- Índice GIN para búsquedas rápidas

**Arrays:**
- Cualquier tipo, multidimensional
- Índice 1-based (primer elemento es [1])
- Índice GIN para búsquedas

**UUID:**
- Identificadores únicos globales
- `gen_random_uuid()` para generar
- Ideal para APIs públicas

**ENUM:**
- Valores predefinidos
- Eficiente internamente
- Difícil de modificar

**Rangos:**
- Intervalos de valores
- Operadores: `@>`, `<@`, `&&`
- Útil para reservas, disponibilidad

**Próximo módulo:** Índices y optimización de performance

---

## Recursos

- **[JSON Types](https://www.postgresql.org/docs/current/datatype-json.html)** - Documentación oficial
- **[Array Types](https://www.postgresql.org/docs/current/arrays.html)** - Arrays en PostgreSQL
- **[Range Types](https://www.postgresql.org/docs/current/rangetypes.html)** - Tipos de rangos

¡Dominas los tipos avanzados de PostgreSQL! 🎯

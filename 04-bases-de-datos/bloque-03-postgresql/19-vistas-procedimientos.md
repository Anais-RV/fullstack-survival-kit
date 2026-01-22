# Vistas, Funciones y Triggers

> **Automatizar lógica con vistas, funciones almacenadas y triggers**

---

## Vistas (Views)

Una **vista** es una query guardada como tabla virtual.

**Ventajas:**
- ✅ Simplifica queries complejas
- ✅ Seguridad (oculta columnas sensibles)
- ✅ Abstracción (cambia estructura sin afectar código)

### Crear vista simple

```sql
CREATE VIEW vista_productos_activos AS
SELECT id, nombre, precio, stock
FROM productos
WHERE activo = true;

-- Usar como tabla
SELECT * FROM vista_productos_activos;
```

### Vista con JOINs

```sql
CREATE VIEW vista_pedidos_completos AS
SELECT 
    p.id AS pedido_id,
    p.fecha,
    p.total,
    p.estado,
    u.nombre AS cliente,
    u.email
FROM pedidos p
INNER JOIN usuarios u ON p.usuario_id = u.id;

-- Consultar
SELECT * FROM vista_pedidos_completos WHERE estado = 'pendiente';
```

### Vista con agregaciones

```sql
CREATE VIEW vista_ventas_por_producto AS
SELECT 
    prod.id,
    prod.nombre,
    COUNT(i.id) AS veces_vendido,
    SUM(i.cantidad) AS unidades_vendidas,
    SUM(i.cantidad * i.precio) AS ingresos_totales
FROM productos prod
LEFT JOIN items_pedido i ON prod.id = i.producto_id
GROUP BY prod.id, prod.nombre;

-- Usar
SELECT * FROM vista_ventas_por_producto ORDER BY ingresos_totales DESC LIMIT 10;
```

### Modificar vista

```sql
-- Reemplazar vista
CREATE OR REPLACE VIEW vista_productos_activos AS
SELECT id, nombre, precio, stock, categoria_id
FROM productos
WHERE activo = true AND stock > 0;

-- Eliminar vista
DROP VIEW vista_productos_activos;
```

### Vistas actualizables

**Condiciones para UPDATE/INSERT/DELETE en vistas:**
- Una sola tabla base
- Sin DISTINCT, GROUP BY, HAVING, UNION
- Sin agregaciones

```sql
-- Vista simple (actualizable)
CREATE VIEW vista_usuarios_activos AS
SELECT id, nombre, email
FROM usuarios
WHERE activo = true;

-- ✅ Funciona
UPDATE vista_usuarios_activos SET email = 'nuevo@email.com' WHERE id = 1;
INSERT INTO vista_usuarios_activos (nombre, email) VALUES ('Ana', 'ana@email.com');

-- ❌ No funciona (vista con JOIN)
CREATE VIEW vista_pedidos_completos AS
SELECT p.id, p.total, u.nombre
FROM pedidos p
INNER JOIN usuarios u ON p.usuario_id = u.id;

UPDATE vista_pedidos_completos SET total = 200 WHERE id = 1;  -- ERROR
```

---

## Materialized Views

**Materialized View:** Vista que almacena físicamente los datos (caché).

**Diferencia con Vista normal:**
- Vista → Query en tiempo real
- Materialized View → Datos precalculados

### Crear Materialized View

```sql
CREATE MATERIALIZED VIEW mv_reporte_ventas AS
SELECT 
    DATE_TRUNC('month', fecha) AS mes,
    COUNT(*) AS num_pedidos,
    SUM(total) AS ingresos,
    AVG(total) AS ticket_promedio
FROM pedidos
GROUP BY DATE_TRUNC('month', fecha)
ORDER BY mes DESC;

-- Crear índice (para queries rápidas)
CREATE INDEX idx_mv_reporte_ventas_mes ON mv_reporte_ventas(mes);
```

### Refrescar datos

```sql
-- Refrescar datos (bloquea la vista)
REFRESH MATERIALIZED VIEW mv_reporte_ventas;

-- Refrescar sin bloqueo (requiere índice UNIQUE)
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_reporte_ventas;
```

### Cuándo usar Materialized Views

✅ **Usar cuando:**
- Query muy lento (varios segundos)
- Datos no cambian frecuentemente
- Consultas repetitivas (reportes)

❌ **No usar cuando:**
- Necesitas datos en tiempo real
- Datos cambian constantemente

### Ejemplo práctico

```sql
-- Reporte costoso (tarda 5 segundos)
SELECT 
    c.nombre AS categoria,
    COUNT(DISTINCT p.id) AS productos,
    SUM(i.cantidad) AS unidades_vendidas,
    SUM(i.cantidad * i.precio) AS ingresos
FROM categorias c
LEFT JOIN productos p ON c.id = p.categoria_id
LEFT JOIN items_pedido i ON p.id = i.producto_id
LEFT JOIN pedidos ped ON i.pedido_id = ped.id
WHERE ped.fecha >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY c.id, c.nombre;

-- Crear Materialized View (calcular una vez)
CREATE MATERIALIZED VIEW mv_ventas_por_categoria AS
SELECT 
    c.nombre AS categoria,
    COUNT(DISTINCT p.id) AS productos,
    SUM(i.cantidad) AS unidades_vendidas,
    SUM(i.cantidad * i.precio) AS ingresos
FROM categorias c
LEFT JOIN productos p ON c.id = p.categoria_id
LEFT JOIN items_pedido i ON p.id = i.producto_id
LEFT JOIN pedidos ped ON i.pedido_id = ped.id
WHERE ped.fecha >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY c.id, c.nombre;

-- Consultar (instantáneo)
SELECT * FROM mv_ventas_por_categoria ORDER BY ingresos DESC;

-- Refrescar diariamente (cron job)
REFRESH MATERIALIZED VIEW mv_ventas_por_categoria;
```

---

## Funciones (Functions)

**Funciones** encapsulan lógica SQL reutilizable.

### Función simple

```sql
CREATE OR REPLACE FUNCTION calcular_descuento(precio NUMERIC, porcentaje INTEGER)
RETURNS NUMERIC AS $$
BEGIN
    RETURN precio * (1 - porcentaje::NUMERIC / 100);
END;
$$ LANGUAGE plpgsql;

-- Usar
SELECT calcular_descuento(100, 20);  -- Resultado: 80
```

### Función con query

```sql
CREATE OR REPLACE FUNCTION obtener_stock(producto_id INTEGER)
RETURNS INTEGER AS $$
DECLARE
    stock_actual INTEGER;
BEGIN
    SELECT stock INTO stock_actual
    FROM productos
    WHERE id = producto_id;
    
    RETURN stock_actual;
END;
$$ LANGUAGE plpgsql;

-- Usar
SELECT obtener_stock(5);
```

### Función que devuelve tabla

```sql
CREATE OR REPLACE FUNCTION productos_baratos(precio_maximo NUMERIC)
RETURNS TABLE(id INTEGER, nombre VARCHAR, precio NUMERIC) AS $$
BEGIN
    RETURN QUERY
    SELECT p.id, p.nombre, p.precio
    FROM productos p
    WHERE p.precio <= precio_maximo
    ORDER BY p.precio;
END;
$$ LANGUAGE plpgsql;

-- Usar
SELECT * FROM productos_baratos(50);
```

### Función con lógica compleja

```sql
CREATE OR REPLACE FUNCTION crear_pedido(
    p_usuario_id INTEGER,
    p_producto_id INTEGER,
    p_cantidad INTEGER
)
RETURNS INTEGER AS $$
DECLARE
    v_pedido_id INTEGER;
    v_precio NUMERIC;
    v_stock INTEGER;
BEGIN
    -- Verificar stock
    SELECT precio, stock INTO v_precio, v_stock
    FROM productos
    WHERE id = p_producto_id;
    
    IF v_stock < p_cantidad THEN
        RAISE EXCEPTION 'Stock insuficiente (disponible: %)', v_stock;
    END IF;
    
    -- Crear pedido
    INSERT INTO pedidos (usuario_id, total, estado)
    VALUES (p_usuario_id, v_precio * p_cantidad, 'pendiente')
    RETURNING id INTO v_pedido_id;
    
    -- Agregar item
    INSERT INTO items_pedido (pedido_id, producto_id, cantidad, precio)
    VALUES (v_pedido_id, p_producto_id, p_cantidad, v_precio);
    
    -- Actualizar stock
    UPDATE productos
    SET stock = stock - p_cantidad
    WHERE id = p_producto_id;
    
    -- Retornar ID
    RETURN v_pedido_id;
    
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Error: %', SQLERRM;
        RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Usar
SELECT crear_pedido(1, 5, 2);
```

### Función con múltiples salidas

```sql
CREATE OR REPLACE FUNCTION estadisticas_pedido(pedido_id INTEGER)
RETURNS TABLE(
    total_items INTEGER,
    total_productos INTEGER,
    monto_total NUMERIC
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        COUNT(*)::INTEGER AS total_items,
        SUM(cantidad)::INTEGER AS total_productos,
        SUM(cantidad * precio) AS monto_total
    FROM items_pedido
    WHERE items_pedido.pedido_id = estadisticas_pedido.pedido_id;
END;
$$ LANGUAGE plpgsql;

-- Usar
SELECT * FROM estadisticas_pedido(123);
```

---

## Triggers

**Triggers** ejecutan código automáticamente en eventos (INSERT, UPDATE, DELETE).

### Sintaxis

```sql
-- 1. Crear función trigger
CREATE OR REPLACE FUNCTION funcion_trigger()
RETURNS TRIGGER AS $$
BEGIN
    -- Lógica
    RETURN NEW;  -- O RETURN OLD o RETURN NULL
END;
$$ LANGUAGE plpgsql;

-- 2. Asociar trigger a tabla
CREATE TRIGGER nombre_trigger
    BEFORE INSERT OR UPDATE ON tabla
    FOR EACH ROW
    EXECUTE FUNCTION funcion_trigger();
```

### Trigger BEFORE INSERT

```sql
-- Actualizar timestamp automáticamente
CREATE OR REPLACE FUNCTION actualizar_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.actualizado_en = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_actualizar_timestamp
    BEFORE UPDATE ON usuarios
    FOR EACH ROW
    EXECUTE FUNCTION actualizar_timestamp();

-- Ahora al hacer UPDATE, actualizado_en se actualiza solo
UPDATE usuarios SET nombre = 'Ana García' WHERE id = 1;
```

### Trigger para auditoría

```sql
-- Tabla de auditoría
CREATE TABLE auditoria_productos (
    id SERIAL PRIMARY KEY,
    producto_id INTEGER,
    accion VARCHAR(10),
    usuario VARCHAR(100),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    datos_antiguos JSONB,
    datos_nuevos JSONB
);

-- Función trigger
CREATE OR REPLACE FUNCTION auditar_productos()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO auditoria_productos (producto_id, accion, usuario, datos_nuevos)
        VALUES (NEW.id, 'INSERT', current_user, to_jsonb(NEW));
        
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO auditoria_productos (producto_id, accion, usuario, datos_antiguos, datos_nuevos)
        VALUES (NEW.id, 'UPDATE', current_user, to_jsonb(OLD), to_jsonb(NEW));
        
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO auditoria_productos (producto_id, accion, usuario, datos_antiguos)
        VALUES (OLD.id, 'DELETE', current_user, to_jsonb(OLD));
        RETURN OLD;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Crear trigger
CREATE TRIGGER trigger_auditar_productos
    AFTER INSERT OR UPDATE OR DELETE ON productos
    FOR EACH ROW
    EXECUTE FUNCTION auditar_productos();

-- Ahora toda modificación queda registrada
UPDATE productos SET precio = 150 WHERE id = 1;
SELECT * FROM auditoria_productos;
```

### Trigger para validación

```sql
-- Validar email antes de insertar
CREATE OR REPLACE FUNCTION validar_email()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' THEN
        RAISE EXCEPTION 'Email inválido: %', NEW.email;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_validar_email
    BEFORE INSERT OR UPDATE ON usuarios
    FOR EACH ROW
    EXECUTE FUNCTION validar_email();

-- Ahora inserts con email inválido fallan
INSERT INTO usuarios (nombre, email) VALUES ('Ana', 'email-invalido');  -- ERROR
```

### Trigger para mantener totales

```sql
-- Actualizar total de pedido automáticamente
CREATE OR REPLACE FUNCTION actualizar_total_pedido()
RETURNS TRIGGER AS $$
BEGIN
    -- Al insertar/actualizar/eliminar item
    UPDATE pedidos
    SET total = (
        SELECT COALESCE(SUM(cantidad * precio), 0)
        FROM items_pedido
        WHERE pedido_id = COALESCE(NEW.pedido_id, OLD.pedido_id)
    )
    WHERE id = COALESCE(NEW.pedido_id, OLD.pedido_id);
    
    IF TG_OP = 'DELETE' THEN
        RETURN OLD;
    ELSE
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_actualizar_total
    AFTER INSERT OR UPDATE OR DELETE ON items_pedido
    FOR EACH ROW
    EXECUTE FUNCTION actualizar_total_pedido();

-- Ahora al modificar items, el total se actualiza solo
INSERT INTO items_pedido (pedido_id, producto_id, cantidad, precio)
VALUES (1, 5, 2, 50);  -- pedidos.total se actualiza a 100
```

### Gestionar triggers

```sql
-- Listar triggers
SELECT 
    trigger_name,
    event_manipulation AS event,
    event_object_table AS table_name
FROM information_schema.triggers
WHERE trigger_schema = 'public';

-- Desactivar trigger
ALTER TABLE productos DISABLE TRIGGER trigger_auditar_productos;

-- Activar trigger
ALTER TABLE productos ENABLE TRIGGER trigger_auditar_productos;

-- Eliminar trigger
DROP TRIGGER trigger_auditar_productos ON productos;
```

---

## PL/pgSQL Básico

**PL/pgSQL** es el lenguaje de programación de PostgreSQL.

### Variables

```sql
CREATE OR REPLACE FUNCTION ejemplo_variables()
RETURNS VOID AS $$
DECLARE
    v_numero INTEGER := 10;
    v_texto VARCHAR := 'Hola';
    v_fecha DATE := CURRENT_DATE;
BEGIN
    RAISE NOTICE 'Número: %, Texto: %, Fecha: %', v_numero, v_texto, v_fecha;
END;
$$ LANGUAGE plpgsql;
```

### Condicionales (IF)

```sql
CREATE OR REPLACE FUNCTION categoria_precio(precio NUMERIC)
RETURNS VARCHAR AS $$
BEGIN
    IF precio < 50 THEN
        RETURN 'Barato';
    ELSIF precio BETWEEN 50 AND 200 THEN
        RETURN 'Medio';
    ELSE
        RETURN 'Caro';
    END IF;
END;
$$ LANGUAGE plpgsql;

SELECT categoria_precio(150);  -- 'Medio'
```

### Bucles (LOOP, WHILE, FOR)

```sql
-- FOR loop con rango
CREATE OR REPLACE FUNCTION sumar_hasta(n INTEGER)
RETURNS INTEGER AS $$
DECLARE
    suma INTEGER := 0;
    i INTEGER;
BEGIN
    FOR i IN 1..n LOOP
        suma := suma + i;
    END LOOP;
    
    RETURN suma;
END;
$$ LANGUAGE plpgsql;

-- FOR loop con query
CREATE OR REPLACE FUNCTION listar_productos()
RETURNS VOID AS $$
DECLARE
    prod RECORD;
BEGIN
    FOR prod IN SELECT id, nombre, precio FROM productos LOOP
        RAISE NOTICE 'ID: %, Nombre: %, Precio: %', prod.id, prod.nombre, prod.precio;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- WHILE loop
CREATE OR REPLACE FUNCTION factorial(n INTEGER)
RETURNS INTEGER AS $$
DECLARE
    resultado INTEGER := 1;
    i INTEGER := 1;
BEGIN
    WHILE i <= n LOOP
        resultado := resultado * i;
        i := i + 1;
    END LOOP;
    
    RETURN resultado;
END;
$$ LANGUAGE plpgsql;
```

### Excepciones

```sql
CREATE OR REPLACE FUNCTION dividir(a NUMERIC, b NUMERIC)
RETURNS NUMERIC AS $$
BEGIN
    IF b = 0 THEN
        RAISE EXCEPTION 'División por cero';
    END IF;
    
    RETURN a / b;
    
EXCEPTION
    WHEN division_by_zero THEN
        RAISE NOTICE 'Error: No se puede dividir por cero';
        RETURN NULL;
    WHEN OTHERS THEN
        RAISE NOTICE 'Error inesperado: %', SQLERRM;
        RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

---

## Caso práctico completo

### Sistema de inventario con triggers

```sql
-- Tablas
CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(200),
    precio NUMERIC(10, 2),
    stock INTEGER DEFAULT 0,
    stock_minimo INTEGER DEFAULT 10,
    activo BOOLEAN DEFAULT true,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE movimientos_stock (
    id SERIAL PRIMARY KEY,
    producto_id INTEGER REFERENCES productos(id),
    tipo VARCHAR(20) CHECK (tipo IN ('entrada', 'salida')),
    cantidad INTEGER,
    stock_anterior INTEGER,
    stock_nuevo INTEGER,
    motivo VARCHAR(200),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE alertas_stock (
    id SERIAL PRIMARY KEY,
    producto_id INTEGER REFERENCES productos(id),
    mensaje TEXT,
    resuelta BOOLEAN DEFAULT false,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Vista de productos con bajo stock
CREATE VIEW vista_productos_bajo_stock AS
SELECT 
    id,
    nombre,
    stock,
    stock_minimo,
    stock - stock_minimo AS diferencia
FROM productos
WHERE stock <= stock_minimo AND activo = true;

-- Función para registrar movimiento
CREATE OR REPLACE FUNCTION registrar_movimiento_stock()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.stock <> NEW.stock THEN
        INSERT INTO movimientos_stock (
            producto_id,
            tipo,
            cantidad,
            stock_anterior,
            stock_nuevo,
            motivo
        ) VALUES (
            NEW.id,
            CASE WHEN NEW.stock > OLD.stock THEN 'entrada' ELSE 'salida' END,
            ABS(NEW.stock - OLD.stock),
            OLD.stock,
            NEW.stock,
            'Actualización manual'
        );
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Función para alertas de stock bajo
CREATE OR REPLACE FUNCTION alertar_stock_bajo()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.stock <= NEW.stock_minimo AND OLD.stock > OLD.stock_minimo THEN
        INSERT INTO alertas_stock (producto_id, mensaje)
        VALUES (
            NEW.id,
            format('Producto "%s" tiene stock bajo: %s unidades (mínimo: %s)',
                   NEW.nombre, NEW.stock, NEW.stock_minimo)
        );
        
        RAISE NOTICE 'ALERTA: Stock bajo en producto %', NEW.nombre;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Función para actualizar timestamp
CREATE OR REPLACE FUNCTION actualizar_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.actualizado_en = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Triggers
CREATE TRIGGER trigger_movimientos_stock
    AFTER UPDATE OF stock ON productos
    FOR EACH ROW
    WHEN (OLD.stock IS DISTINCT FROM NEW.stock)
    EXECUTE FUNCTION registrar_movimiento_stock();

CREATE TRIGGER trigger_alertas_stock
    AFTER UPDATE OF stock ON productos
    FOR EACH ROW
    EXECUTE FUNCTION alertar_stock_bajo();

CREATE TRIGGER trigger_timestamp_productos
    BEFORE UPDATE ON productos
    FOR EACH ROW
    EXECUTE FUNCTION actualizar_timestamp();

-- Función para ajustar stock
CREATE OR REPLACE FUNCTION ajustar_stock(
    p_producto_id INTEGER,
    p_cantidad INTEGER,
    p_tipo VARCHAR,
    p_motivo VARCHAR DEFAULT NULL
)
RETURNS VOID AS $$
DECLARE
    v_stock_actual INTEGER;
BEGIN
    -- Obtener stock actual con lock
    SELECT stock INTO v_stock_actual
    FROM productos
    WHERE id = p_producto_id
    FOR UPDATE;
    
    IF NOT FOUND THEN
        RAISE EXCEPTION 'Producto % no existe', p_producto_id;
    END IF;
    
    -- Actualizar stock
    IF p_tipo = 'entrada' THEN
        UPDATE productos
        SET stock = stock + p_cantidad
        WHERE id = p_producto_id;
        
    ELSIF p_tipo = 'salida' THEN
        IF v_stock_actual < p_cantidad THEN
            RAISE EXCEPTION 'Stock insuficiente (disponible: %)', v_stock_actual;
        END IF;
        
        UPDATE productos
        SET stock = stock - p_cantidad
        WHERE id = p_producto_id;
    ELSE
        RAISE EXCEPTION 'Tipo inválido: % (debe ser entrada/salida)', p_tipo;
    END IF;
    
    -- Actualizar motivo en último movimiento
    IF p_motivo IS NOT NULL THEN
        UPDATE movimientos_stock
        SET motivo = p_motivo
        WHERE id = (SELECT MAX(id) FROM movimientos_stock WHERE producto_id = p_producto_id);
    END IF;
    
    RAISE NOTICE 'Stock ajustado correctamente';
END;
$$ LANGUAGE plpgsql;

-- Usar el sistema
INSERT INTO productos (nombre, precio, stock, stock_minimo) VALUES
    ('Laptop', 1200, 50, 10),
    ('Mouse', 25, 100, 20);

-- Ajustar stock (trigger registra movimiento automáticamente)
SELECT ajustar_stock(1, 45, 'salida', 'Venta masiva');

-- Ver movimientos
SELECT * FROM movimientos_stock;

-- Ver productos con bajo stock
SELECT * FROM vista_productos_bajo_stock;

-- Ver alertas
SELECT * FROM alertas_stock WHERE resuelta = false;
```

---

## Resumen

**Vistas:**
- Query guardada como tabla virtual
- `CREATE VIEW` - Datos en tiempo real
- `CREATE MATERIALIZED VIEW` - Datos precalculados

**Funciones:**
```sql
CREATE FUNCTION nombre(parametros) RETURNS tipo AS $$
BEGIN
    -- lógica
    RETURN valor;
END;
$$ LANGUAGE plpgsql;
```

**Triggers:**
```sql
CREATE TRIGGER nombre
    BEFORE/AFTER INSERT/UPDATE/DELETE ON tabla
    FOR EACH ROW
    EXECUTE FUNCTION funcion_trigger();
```

**Uso común:**
- Vistas → Simplificar queries
- Materialized Views → Reportes pesados
- Funciones → Lógica reutilizable
- Triggers → Automatización (auditoría, validación, totales)

**Próximo módulo:** PostgreSQL con Python (psycopg2)

---

## Recursos

- **[Views](https://www.postgresql.org/docs/current/sql-createview.html)** - Documentación oficial
- **[Functions](https://www.postgresql.org/docs/current/xfunc.html)** - Funciones en PostgreSQL
- **[Triggers](https://www.postgresql.org/docs/current/triggers.html)** - Triggers completo
- **[PL/pgSQL](https://www.postgresql.org/docs/current/plpgsql.html)** - Lenguaje procedural

¡Automatiza toda tu lógica de negocio! ⚙️

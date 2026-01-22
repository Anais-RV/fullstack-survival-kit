# Transacciones y Propiedades ACID

> **Garantizar integridad de datos con transacciones**

---

## ¿Qué es una transacción?

Una **transacción** es un conjunto de operaciones que se ejecutan como **una sola unidad**.

**Principio todo o nada:**
- ✅ Si todas las operaciones tienen éxito → COMMIT
- ❌ Si alguna falla → ROLLBACK (revertir todo)

**Analogía:**  
Como transferir dinero entre cuentas bancarias:
1. Restar $100 de cuenta A
2. Sumar $100 a cuenta B

Si el paso 2 falla, el paso 1 debe revertirse (no puede quedar inconsistente).

---

## Propiedades ACID

**ACID** garantiza confiabilidad de transacciones.

### A - Atomicity (Atomicidad)

**Definición:** Todo o nada.

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
COMMIT;  -- ✅ Ambas se ejecutan

-- Si falla una:
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;  -- ✅ Éxito
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 999;  -- ❌ Error (no existe)
-- PostgreSQL revierte automáticamente ambas
```

### C - Consistency (Consistencia)

**Definición:** La BD siempre está en estado válido.

```sql
-- Constraint garantiza consistencia
CREATE TABLE cuentas (
    id SERIAL PRIMARY KEY,
    saldo NUMERIC(10, 2) CHECK (saldo >= 0)  -- No puede ser negativo
);

BEGIN;
UPDATE cuentas SET saldo = saldo - 200 WHERE id = 1;
-- ERROR: violates check constraint "cuentas_saldo_check"
-- Transacción revertida → BD sigue consistente
```

### I - Isolation (Aislamiento)

**Definición:** Transacciones concurrentes no se interfieren.

```sql
-- Usuario A
BEGIN;
UPDATE productos SET stock = stock - 1 WHERE id = 1;
-- (aún no commit)

-- Usuario B (en paralelo)
BEGIN;
SELECT stock FROM productos WHERE id = 1;
-- Ve el valor ANTES de la actualización de A (aislamiento)
```

### D - Durability (Durabilidad)

**Definición:** Después de COMMIT, los cambios son permanentes.

```sql
BEGIN;
INSERT INTO pedidos (usuario_id, total) VALUES (1, 150.00);
COMMIT;
-- Incluso si el servidor se cae AHORA, el pedido persiste
```

---

## Sintaxis de transacciones

### BEGIN / COMMIT / ROLLBACK

```sql
-- Iniciar transacción
BEGIN;
-- O: START TRANSACTION;

-- Operaciones
INSERT INTO usuarios (nombre, email) VALUES ('Ana', 'ana@email.com');
UPDATE productos SET stock = stock - 1 WHERE id = 5;

-- Confirmar cambios
COMMIT;

-- O revertir
ROLLBACK;
```

### Ejemplo: Transferencia bancaria

```sql
BEGIN;

-- Restar de cuenta origen
UPDATE cuentas 
SET saldo = saldo - 100 
WHERE id = 1 AND saldo >= 100;

-- Verificar que se actualizó
IF NOT FOUND THEN
    ROLLBACK;
    RAISE EXCEPTION 'Saldo insuficiente';
END IF;

-- Sumar a cuenta destino
UPDATE cuentas 
SET saldo = saldo + 100 
WHERE id = 2;

COMMIT;
```

---

## Niveles de aislamiento

PostgreSQL soporta 4 niveles de aislamiento (estándar SQL).

### 1. Read Uncommitted

**Comportamiento:** Lee cambios no confirmados de otras transacciones.

**PostgreSQL:** No implementado (se comporta como Read Committed).

### 2. Read Committed (por defecto)

**Comportamiento:** Solo ve cambios confirmados.

```sql
-- Transacción A
BEGIN;
UPDATE productos SET precio = 100 WHERE id = 1;
-- (no commit aún)

-- Transacción B
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT precio FROM productos WHERE id = 1;
-- Resultado: 50 (valor anterior, A no hizo commit)
COMMIT;

-- Transacción A
COMMIT;

-- Transacción B (nueva)
BEGIN;
SELECT precio FROM productos WHERE id = 1;
-- Resultado: 100 (ahora sí ve el cambio)
```

**Problema: Non-repeatable Read**

```sql
-- Transacción A
BEGIN;
SELECT precio FROM productos WHERE id = 1;  -- 50

-- Transacción B (en paralelo)
BEGIN;
UPDATE productos SET precio = 100 WHERE id = 1;
COMMIT;

-- Transacción A (continúa)
SELECT precio FROM productos WHERE id = 1;  -- 100 (cambió!)
COMMIT;
```

### 3. Repeatable Read

**Comportamiento:** Ve snapshot al inicio de la transacción.

```sql
-- Transacción A
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT precio FROM productos WHERE id = 1;  -- 50

-- Transacción B
BEGIN;
UPDATE productos SET precio = 100 WHERE id = 1;
COMMIT;

-- Transacción A (continúa)
SELECT precio FROM productos WHERE id = 1;  -- 50 (no ve el cambio de B)
COMMIT;

-- Ahora sí ve 100
SELECT precio FROM productos WHERE id = 1;  -- 100
```

**Problema: Phantom Reads (en teoría)**

```sql
-- Transacción A
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM productos WHERE categoria = 'Electrónica';  -- 10

-- Transacción B
INSERT INTO productos (nombre, categoria) VALUES ('Laptop', 'Electrónica');
COMMIT;

-- Transacción A (continúa)
SELECT COUNT(*) FROM productos WHERE categoria = 'Electrónica';  -- 10 (no ve el nuevo)
```

**Nota:** PostgreSQL previene phantom reads en Repeatable Read (implementación MVCC).

### 4. Serializable

**Comportamiento:** Máximo aislamiento (como si transacciones fueran secuenciales).

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Operaciones...
COMMIT;
```

**Ventaja:** Previene todas las anomalías.  
**Desventaja:** Más lento, más conflictos (serialization failures).

### Comparación de niveles

| Nivel | Dirty Read | Non-repeatable Read | Phantom Read | Performance |
|---|---|---|---|---|
| **Read Uncommitted** | Posible | Posible | Posible | ⚡⚡⚡ |
| **Read Committed** | No | Posible | Posible | ⚡⚡ |
| **Repeatable Read** | No | No | No (PostgreSQL) | ⚡ |
| **Serializable** | No | No | No | 💤 |

**Recomendación:** Read Committed (por defecto) para la mayoría de casos.

---

## SAVEPOINT

**SAVEPOINT** permite revertir parcialmente una transacción.

```sql
BEGIN;

-- Operación 1
INSERT INTO usuarios (nombre) VALUES ('Ana');

-- Crear punto de guardado
SAVEPOINT sp1;

-- Operación 2
INSERT INTO usuarios (nombre) VALUES ('Bob');

-- Crear otro punto
SAVEPOINT sp2;

-- Operación 3
INSERT INTO usuarios (nombre) VALUES ('Carlos');

-- Revertir a sp2 (elimina solo Carlos)
ROLLBACK TO sp2;

-- Revertir a sp1 (elimina Bob y Carlos)
ROLLBACK TO sp1;

-- Confirmar (solo Ana se guarda)
COMMIT;
```

### Ejemplo práctico

```sql
BEGIN;

-- Crear pedido
INSERT INTO pedidos (usuario_id, total) 
VALUES (1, 0) 
RETURNING id INTO pedido_id;

SAVEPOINT items_inicio;

-- Agregar items
INSERT INTO items_pedido (pedido_id, producto_id, cantidad, precio)
VALUES (pedido_id, 1, 2, 50);

-- Error en siguiente item
BEGIN
    INSERT INTO items_pedido (pedido_id, producto_id, cantidad, precio)
    VALUES (pedido_id, 999, 1, 100);  -- Producto no existe
EXCEPTION
    WHEN foreign_key_violation THEN
        ROLLBACK TO items_inicio;  -- Revertir items, mantener pedido
        RAISE NOTICE 'Item inválido, pedido sin items';
END;

-- Actualizar total
UPDATE pedidos SET total = 100 WHERE id = pedido_id;

COMMIT;
```

---

## Locks (Bloqueos)

PostgreSQL usa **MVCC** (Multi-Version Concurrency Control), pero a veces necesitas locks explícitos.

### Tipos de locks

**1. Row-level locks**

```sql
-- SELECT FOR UPDATE (bloqueo exclusivo)
BEGIN;
SELECT * FROM productos WHERE id = 1 FOR UPDATE;
-- Otras transacciones esperan hasta COMMIT/ROLLBACK
UPDATE productos SET stock = stock - 1 WHERE id = 1;
COMMIT;

-- SELECT FOR SHARE (bloqueo compartido)
BEGIN;
SELECT * FROM productos WHERE id = 1 FOR SHARE;
-- Otras transacciones pueden leer, pero no modificar
COMMIT;
```

**2. Table-level locks**

```sql
-- Bloqueo exclusivo de tabla
BEGIN;
LOCK TABLE productos IN EXCLUSIVE MODE;
-- Nadie más puede acceder a la tabla
-- (raro, solo para mantenimiento)
COMMIT;
```

### Deadlocks

**Deadlock:** Dos transacciones esperan mutuamente.

```sql
-- Transacción A
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
-- Espera lock en id=2

-- Transacción B (en paralelo)
BEGIN;
UPDATE cuentas SET saldo = saldo + 50 WHERE id = 2;
-- Espera lock en id=1

-- Transacción A (continúa)
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;  -- Espera a B

-- Transacción B (continúa)
UPDATE cuentas SET saldo = saldo - 50 WHERE id = 1;  -- Espera a A

-- ❌ DEADLOCK! PostgreSQL detecta y aborta una transacción
```

**Prevenir deadlocks:**
- Siempre acceder a filas en el mismo orden
- Usar timeouts
- Mantener transacciones cortas

```sql
-- ✅ Orden consistente (siempre id menor primero)
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = LEAST(1, 2);
UPDATE cuentas SET saldo = saldo + 100 WHERE id = GREATEST(1, 2);
COMMIT;
```

---

## Caso práctico: E-commerce

### Crear pedido con items (transacción completa)

```sql
CREATE OR REPLACE FUNCTION crear_pedido(
    p_usuario_id INTEGER,
    p_items JSONB  -- [{"producto_id": 1, "cantidad": 2}, ...]
)
RETURNS INTEGER AS $$
DECLARE
    v_pedido_id INTEGER;
    v_item JSONB;
    v_producto RECORD;
    v_total NUMERIC := 0;
BEGIN
    -- Iniciar transacción implícita
    
    -- 1. Crear pedido
    INSERT INTO pedidos (usuario_id, total, estado)
    VALUES (p_usuario_id, 0, 'pendiente')
    RETURNING id INTO v_pedido_id;
    
    -- 2. Procesar items
    FOR v_item IN SELECT * FROM jsonb_array_elements(p_items)
    LOOP
        -- Obtener producto con lock
        SELECT id, precio, stock INTO v_producto
        FROM productos
        WHERE id = (v_item->>'producto_id')::INTEGER
        FOR UPDATE;
        
        -- Verificar stock
        IF v_producto.stock < (v_item->>'cantidad')::INTEGER THEN
            RAISE EXCEPTION 'Stock insuficiente para producto %', v_producto.id;
        END IF;
        
        -- Insertar item
        INSERT INTO items_pedido (pedido_id, producto_id, cantidad, precio)
        VALUES (
            v_pedido_id,
            v_producto.id,
            (v_item->>'cantidad')::INTEGER,
            v_producto.precio
        );
        
        -- Actualizar stock
        UPDATE productos
        SET stock = stock - (v_item->>'cantidad')::INTEGER
        WHERE id = v_producto.id;
        
        -- Acumular total
        v_total := v_total + (v_producto.precio * (v_item->>'cantidad')::INTEGER);
    END LOOP;
    
    -- 3. Actualizar total del pedido
    UPDATE pedidos
    SET total = v_total
    WHERE id = v_pedido_id;
    
    -- 4. Retornar ID
    RETURN v_pedido_id;
    
EXCEPTION
    WHEN OTHERS THEN
        -- Revertir automáticamente
        RAISE NOTICE 'Error al crear pedido: %', SQLERRM;
        RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Usar función
SELECT crear_pedido(
    1,  -- usuario_id
    '[
        {"producto_id": 1, "cantidad": 2},
        {"producto_id": 3, "cantidad": 1}
    ]'::JSONB
);
```

### Cancelar pedido (revertir stock)

```sql
CREATE OR REPLACE FUNCTION cancelar_pedido(p_pedido_id INTEGER)
RETURNS BOOLEAN AS $$
DECLARE
    v_item RECORD;
BEGIN
    -- Verificar estado
    IF NOT EXISTS (
        SELECT 1 FROM pedidos 
        WHERE id = p_pedido_id AND estado IN ('pendiente', 'pagado')
    ) THEN
        RAISE EXCEPTION 'Pedido no puede ser cancelado';
    END IF;
    
    -- Revertir stock de cada item
    FOR v_item IN 
        SELECT producto_id, cantidad 
        FROM items_pedido 
        WHERE pedido_id = p_pedido_id
    LOOP
        UPDATE productos
        SET stock = stock + v_item.cantidad
        WHERE id = v_item.producto_id;
    END LOOP;
    
    -- Actualizar estado
    UPDATE pedidos
    SET estado = 'cancelado'
    WHERE id = p_pedido_id;
    
    RETURN TRUE;
    
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Error al cancelar pedido: %', SQLERRM;
        RETURN FALSE;
END;
$$ LANGUAGE plpgsql;

-- Usar función
SELECT cancelar_pedido(123);
```

---

## Transacciones en psycopg2 (Python)

```python
import psycopg2

# Conectar
conn = psycopg2.connect(
    host="localhost",
    database="ecommerce",
    user="app_user",
    password="password"
)

# Transacción manual
try:
    cur = conn.cursor()
    
    # Iniciar transacción (implícito)
    cur.execute("UPDATE cuentas SET saldo = saldo - 100 WHERE id = %s", (1,))
    cur.execute("UPDATE cuentas SET saldo = saldo + 100 WHERE id = %s", (2,))
    
    # Confirmar
    conn.commit()
    print("Transferencia exitosa")
    
except Exception as e:
    # Revertir
    conn.rollback()
    print(f"Error: {e}")
    
finally:
    cur.close()
    conn.close()
```

### Context manager (recomendado)

```python
import psycopg2

conn = psycopg2.connect(...)

try:
    with conn:  # Auto-commit/rollback
        with conn.cursor() as cur:
            cur.execute("UPDATE cuentas SET saldo = saldo - 100 WHERE id = %s", (1,))
            cur.execute("UPDATE cuentas SET saldo = saldo + 100 WHERE id = %s", (2,))
            # Si llega aquí: commit automático
            # Si hay excepción: rollback automático
            
except psycopg2.Error as e:
    print(f"Error: {e}")
    
finally:
    conn.close()
```

---

## Monitoreo de transacciones

### Ver transacciones activas

```sql
SELECT 
    pid,
    usename,
    state,
    query,
    xact_start AS transaction_start,
    now() - xact_start AS duration
FROM pg_stat_activity
WHERE state = 'active'
  AND xact_start IS NOT NULL
ORDER BY duration DESC;
```

### Ver locks activos

```sql
SELECT 
    l.pid,
    l.relation::regclass AS table_name,
    l.mode,
    l.granted,
    a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE NOT l.granted  -- Solo locks esperando
ORDER BY l.pid;
```

### Terminar transacción bloqueada

```sql
-- Terminar proceso
SELECT pg_terminate_backend(12345);  -- Reemplazar con PID

-- Cancelar query (más suave)
SELECT pg_cancel_backend(12345);
```

---

## Mejores prácticas

✅ **Transacciones cortas**
```sql
-- ❌ Lento (lock largo)
BEGIN;
SELECT * FROM productos;  -- Millones de filas
-- (procesamiento lento)
UPDATE productos SET stock = stock - 1 WHERE id = 1;
COMMIT;

-- ✅ Rápido
SELECT * FROM productos;  -- Sin transacción
-- (procesamiento)
BEGIN;
UPDATE productos SET stock = stock - 1 WHERE id = 1;
COMMIT;
```

✅ **Usar Read Committed (por defecto)**
```sql
-- Solo cambiar si realmente necesitas:
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

✅ **SAVEPOINT para operaciones complejas**
```sql
BEGIN;
-- Operación crítica
SAVEPOINT sp1;
-- Operación que puede fallar
-- Si falla: ROLLBACK TO sp1
COMMIT;
```

✅ **SELECT FOR UPDATE para evitar race conditions**
```sql
BEGIN;
SELECT stock FROM productos WHERE id = 1 FOR UPDATE;
-- Ahora seguro de que nadie más modifica
IF stock > 0 THEN
    UPDATE productos SET stock = stock - 1 WHERE id = 1;
END IF;
COMMIT;
```

❌ **No hacer I/O en transacciones**
```python
# ❌ Malo
conn.begin()
cur.execute("UPDATE productos SET stock = stock - 1 WHERE id = 1")
enviar_email_lento()  # Mantiene lock!
conn.commit()

# ✅ Bueno
conn.begin()
cur.execute("UPDATE productos SET stock = stock - 1 WHERE id = 1")
conn.commit()
enviar_email_lento()  # Después del commit
```

❌ **No olvidar COMMIT/ROLLBACK**
```sql
BEGIN;
UPDATE productos SET precio = 100 WHERE id = 1;
-- ¡Olvidé COMMIT!
-- Otros usuarios esperan indefinidamente
```

---

## Resumen

**ACID:**
- **A**tomicity - Todo o nada
- **C**onsistency - Siempre estado válido
- **I**solation - Transacciones no interfieren
- **D**urability - Cambios permanentes

**Transacciones:**
```sql
BEGIN;
-- operaciones
COMMIT;  -- o ROLLBACK
```

**Niveles de aislamiento:**
- Read Committed (por defecto, recomendado)
- Repeatable Read (snapshot)
- Serializable (máximo aislamiento)

**SAVEPOINT:**
```sql
SAVEPOINT nombre;
ROLLBACK TO nombre;
```

**Locks:**
- `SELECT ... FOR UPDATE` - Exclusivo
- `SELECT ... FOR SHARE` - Compartido

**Próximo módulo:** Vistas, funciones y triggers

---

## Recursos

- **[Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)** - Tutorial oficial
- **[ACID](https://en.wikipedia.org/wiki/ACID)** - Conceptos teóricos
- **[MVCC](https://www.postgresql.org/docs/current/mvcc.html)** - Implementación de PostgreSQL

¡Tus datos están seguros! 🔒

# PostgreSQL con Python

> **Conectar y trabajar con PostgreSQL desde Python usando psycopg2**

---

## ¿Por qué psycopg2?

**psycopg2** es el driver más popular para PostgreSQL en Python.

**Características:**
- ✅ Implementación completa del protocolo PostgreSQL
- ✅ Thread-safe
- ✅ Connection pooling
- ✅ Compatible con Python DB-API 2.0
- ✅ Usado por Django, Flask-SQLAlchemy, etc.

---

## Instalación

```bash
# Instalar psycopg2
pip install psycopg2-binary

# O versión de desarrollo (requiere compilador)
pip install psycopg2
```

**Diferencia:**
- `psycopg2-binary` - Precompilado, más fácil
- `psycopg2` - Compilado localmente, mejor performance

---

## Conexión básica

```python
import psycopg2

# Conectar
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    database="ecommerce",
    user="app_user",
    password="password"
)

# Verificar conexión
print("Conectado a PostgreSQL")

# Cerrar
conn.close()
```

### Connection string (URI)

```python
import psycopg2

# Formato: postgresql://usuario:password@host:puerto/database
conn = psycopg2.connect(
    "postgresql://app_user:password@localhost:5432/ecommerce"
)
```

### Variables de entorno

```python
import os
import psycopg2

# .env o variables de sistema
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://localhost/ecommerce")

conn = psycopg2.connect(DATABASE_URL)
```

---

## Ejecutar queries

### SELECT

```python
import psycopg2

conn = psycopg2.connect(...)

# Crear cursor
cur = conn.cursor()

# Ejecutar query
cur.execute("SELECT id, nombre, email FROM usuarios")

# Obtener todos los resultados
rows = cur.fetchall()

for row in rows:
    print(f"ID: {row[0]}, Nombre: {row[1]}, Email: {row[2]}")

# Cerrar cursor y conexión
cur.close()
conn.close()
```

### Métodos de fetch

```python
# fetchone() - Una fila
cur.execute("SELECT * FROM usuarios WHERE id = 1")
row = cur.fetchone()
print(row)  # (1, 'Ana', 'ana@email.com', ...)

# fetchall() - Todas las filas
cur.execute("SELECT * FROM usuarios")
rows = cur.fetchall()
print(len(rows))  # 100

# fetchmany(size) - N filas
cur.execute("SELECT * FROM usuarios")
rows = cur.fetchmany(10)
print(len(rows))  # 10
```

### Iterar sobre cursor

```python
cur.execute("SELECT id, nombre FROM usuarios")

for row in cur:
    print(f"ID: {row[0]}, Nombre: {row[1]}")
```

---

## Queries parametrizadas

**¡NUNCA usar f-strings para SQL!** (SQL Injection)

```python
# ❌ PELIGROSO (SQL Injection)
email = input("Email: ")
cur.execute(f"SELECT * FROM usuarios WHERE email = '{email}'")
# Si email = "' OR '1'='1" → Devuelve todos los usuarios!

# ✅ SEGURO (parametrizado)
email = input("Email: ")
cur.execute("SELECT * FROM usuarios WHERE email = %s", (email,))
```

### Parámetros posicionales

```python
# Un parámetro (tupla con coma)
cur.execute("SELECT * FROM usuarios WHERE email = %s", ("ana@email.com",))

# Múltiples parámetros
cur.execute(
    "SELECT * FROM productos WHERE categoria = %s AND precio <= %s",
    ("Electrónica", 500)
)

# Con IN
ids = [1, 2, 3, 4, 5]
cur.execute("SELECT * FROM productos WHERE id IN %s", (tuple(ids),))
```

### Parámetros nombrados

```python
# Diccionario con %(nombre)s
cur.execute(
    "SELECT * FROM productos WHERE categoria = %(cat)s AND precio <= %(precio)s",
    {"cat": "Electrónica", "precio": 500}
)

# Múltiples queries con mismos parámetros
params = {"email": "ana@email.com"}

cur.execute("SELECT * FROM usuarios WHERE email = %(email)s", params)
cur.execute("SELECT * FROM pedidos WHERE usuario_id = (SELECT id FROM usuarios WHERE email = %(email)s)", params)
```

---

## INSERT, UPDATE, DELETE

### INSERT

```python
# Insertar uno
cur.execute(
    "INSERT INTO usuarios (nombre, email) VALUES (%s, %s)",
    ("Ana García", "ana@email.com")
)

# Insertar con RETURNING
cur.execute(
    "INSERT INTO usuarios (nombre, email) VALUES (%s, %s) RETURNING id",
    ("Bob Smith", "bob@email.com")
)
nuevo_id = cur.fetchone()[0]
print(f"Nuevo ID: {nuevo_id}")

# Confirmar cambios
conn.commit()
```

### INSERT múltiples

```python
# executemany()
usuarios = [
    ("Ana", "ana@email.com"),
    ("Bob", "bob@email.com"),
    ("Carlos", "carlos@email.com")
]

cur.executemany(
    "INSERT INTO usuarios (nombre, email) VALUES (%s, %s)",
    usuarios
)

conn.commit()
print(f"{cur.rowcount} filas insertadas")
```

### UPDATE

```python
# Actualizar
cur.execute(
    "UPDATE usuarios SET email = %s WHERE id = %s",
    ("nuevo@email.com", 1)
)

conn.commit()
print(f"{cur.rowcount} filas actualizadas")
```

### DELETE

```python
# Eliminar
cur.execute("DELETE FROM usuarios WHERE id = %s", (5,))

conn.commit()
print(f"{cur.rowcount} filas eliminadas")
```

---

## Transacciones

### Commit y Rollback

```python
import psycopg2

conn = psycopg2.connect(...)
cur = conn.cursor()

try:
    # Operaciones
    cur.execute("UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1")
    cur.execute("UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2")
    
    # Confirmar
    conn.commit()
    print("Transferencia exitosa")
    
except Exception as e:
    # Revertir en caso de error
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
            cur.execute("UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1")
            cur.execute("UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2")
            # Si llega aquí: commit automático
            # Si excepción: rollback automático
            
except psycopg2.Error as e:
    print(f"Error: {e}")
    
finally:
    conn.close()
```

---

## Tipos de datos PostgreSQL

### Convertir tipos automáticamente

```python
import psycopg2
from psycopg2.extras import Json
from datetime import datetime, date
from decimal import Decimal

cur.execute(
    """
    INSERT INTO productos (nombre, precio, metadata, creado_en)
    VALUES (%s, %s, %s, %s)
    """,
    (
        "Laptop",                    # VARCHAR → str
        Decimal("1299.99"),          # NUMERIC → Decimal
        Json({"ram": 16, "ssd": 512}),  # JSONB → dict
        datetime.now()               # TIMESTAMP → datetime
    )
)
```

### Arrays

```python
# Insertar array
cur.execute(
    "INSERT INTO usuarios (nombre, emails) VALUES (%s, %s)",
    ("Ana", ["ana@email.com", "ana@work.com"])
)

# Leer array
cur.execute("SELECT emails FROM usuarios WHERE id = 1")
emails = cur.fetchone()[0]
print(emails)  # ['ana@email.com', 'ana@work.com']
```

### UUID

```python
from uuid import uuid4

# Generar UUID en Python
nuevo_id = uuid4()

cur.execute(
    "INSERT INTO articulos (id, titulo) VALUES (%s, %s)",
    (nuevo_id, "Mi artículo")
)

# O dejar que PostgreSQL genere
cur.execute(
    "INSERT INTO articulos (titulo) VALUES (%s) RETURNING id",
    ("Otro artículo",)
)
```

---

## Diccionarios en lugar de tuplas

### RealDictCursor

```python
from psycopg2.extras import RealDictCursor

conn = psycopg2.connect(...)

# Cursor que devuelve diccionarios
cur = conn.cursor(cursor_factory=RealDictCursor)

cur.execute("SELECT id, nombre, email FROM usuarios")

for row in cur:
    print(f"ID: {row['id']}, Nombre: {row['nombre']}, Email: {row['email']}")
    # Acceso por clave en lugar de índice

cur.close()
conn.close()
```

### NamedTupleCursor

```python
from psycopg2.extras import NamedTupleCursor

cur = conn.cursor(cursor_factory=NamedTupleCursor)

cur.execute("SELECT id, nombre, email FROM usuarios")

for row in cur:
    print(f"ID: {row.id}, Nombre: {row.nombre}, Email: {row.email}")
    # Acceso por atributo
```

---

## Connection Pool

**Connection Pool** reutiliza conexiones (más eficiente).

```python
from psycopg2 import pool

# Crear pool
connection_pool = pool.SimpleConnectionPool(
    minconn=1,      # Mínimo de conexiones
    maxconn=10,     # Máximo de conexiones
    host="localhost",
    database="ecommerce",
    user="app_user",
    password="password"
)

# Obtener conexión del pool
conn = connection_pool.getconn()

try:
    cur = conn.cursor()
    cur.execute("SELECT * FROM usuarios")
    rows = cur.fetchall()
    print(f"{len(rows)} usuarios")
    
finally:
    cur.close()
    # Devolver conexión al pool (no cerrar)
    connection_pool.putconn(conn)

# Al final de la aplicación
connection_pool.closeall()
```

### ThreadedConnectionPool

```python
from psycopg2 import pool

# Para aplicaciones multi-thread
connection_pool = pool.ThreadedConnectionPool(
    minconn=1,
    maxconn=20,
    host="localhost",
    database="ecommerce",
    user="app_user",
    password="password"
)

# Usar igual que SimpleConnectionPool
```

---

## Caso práctico: API de E-commerce

```python
import psycopg2
from psycopg2.extras import RealDictCursor, Json
from decimal import Decimal
from typing import List, Optional

class Database:
    """Clase para manejar conexiones a PostgreSQL"""
    
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.conn = None
    
    def connect(self):
        """Establecer conexión"""
        self.conn = psycopg2.connect(
            self.connection_string,
            cursor_factory=RealDictCursor
        )
    
    def disconnect(self):
        """Cerrar conexión"""
        if self.conn:
            self.conn.close()
    
    def __enter__(self):
        """Context manager: entrada"""
        self.connect()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Context manager: salida"""
        self.disconnect()

class UsuarioRepository:
    """Repositorio de usuarios"""
    
    def __init__(self, db: Database):
        self.db = db
    
    def crear(self, nombre: str, email: str) -> int:
        """Crear usuario"""
        with self.db.conn.cursor() as cur:
            cur.execute(
                "INSERT INTO usuarios (nombre, email) VALUES (%s, %s) RETURNING id",
                (nombre, email)
            )
            self.db.conn.commit()
            return cur.fetchone()['id']
    
    def obtener(self, usuario_id: int) -> Optional[dict]:
        """Obtener usuario por ID"""
        with self.db.conn.cursor() as cur:
            cur.execute("SELECT * FROM usuarios WHERE id = %s", (usuario_id,))
            return cur.fetchone()
    
    def listar(self, limite: int = 100) -> List[dict]:
        """Listar usuarios"""
        with self.db.conn.cursor() as cur:
            cur.execute("SELECT * FROM usuarios LIMIT %s", (limite,))
            return cur.fetchall()
    
    def actualizar(self, usuario_id: int, nombre: str = None, email: str = None) -> bool:
        """Actualizar usuario"""
        campos = []
        valores = []
        
        if nombre:
            campos.append("nombre = %s")
            valores.append(nombre)
        
        if email:
            campos.append("email = %s")
            valores.append(email)
        
        if not campos:
            return False
        
        valores.append(usuario_id)
        query = f"UPDATE usuarios SET {', '.join(campos)} WHERE id = %s"
        
        with self.db.conn.cursor() as cur:
            cur.execute(query, valores)
            self.db.conn.commit()
            return cur.rowcount > 0
    
    def eliminar(self, usuario_id: int) -> bool:
        """Eliminar usuario"""
        with self.db.conn.cursor() as cur:
            cur.execute("DELETE FROM usuarios WHERE id = %s", (usuario_id,))
            self.db.conn.commit()
            return cur.rowcount > 0

class ProductoRepository:
    """Repositorio de productos"""
    
    def __init__(self, db: Database):
        self.db = db
    
    def crear(self, nombre: str, precio: Decimal, stock: int, metadata: dict = None) -> int:
        """Crear producto"""
        with self.db.conn.cursor() as cur:
            cur.execute(
                """
                INSERT INTO productos (nombre, precio, stock, metadata)
                VALUES (%s, %s, %s, %s)
                RETURNING id
                """,
                (nombre, precio, stock, Json(metadata or {}))
            )
            self.db.conn.commit()
            return cur.fetchone()['id']
    
    def buscar(self, query: str) -> List[dict]:
        """Buscar productos por nombre"""
        with self.db.conn.cursor() as cur:
            cur.execute(
                """
                SELECT id, nombre, precio, stock
                FROM productos
                WHERE nombre ILIKE %s AND activo = true
                ORDER BY nombre
                """,
                (f"%{query}%",)
            )
            return cur.fetchall()
    
    def por_categoria(self, categoria: str) -> List[dict]:
        """Productos por categoría"""
        with self.db.conn.cursor() as cur:
            cur.execute(
                """
                SELECT p.id, p.nombre, p.precio, p.stock
                FROM productos p
                INNER JOIN categorias c ON p.categoria_id = c.id
                WHERE c.nombre = %s AND p.activo = true
                """,
                (categoria,)
            )
            return cur.fetchall()

class PedidoRepository:
    """Repositorio de pedidos"""
    
    def __init__(self, db: Database):
        self.db = db
    
    def crear(self, usuario_id: int, items: List[dict]) -> Optional[int]:
        """
        Crear pedido con items
        
        items = [
            {"producto_id": 1, "cantidad": 2},
            {"producto_id": 3, "cantidad": 1}
        ]
        """
        try:
            with self.db.conn:
                with self.db.conn.cursor() as cur:
                    # 1. Crear pedido
                    cur.execute(
                        "INSERT INTO pedidos (usuario_id, total) VALUES (%s, 0) RETURNING id",
                        (usuario_id,)
                    )
                    pedido_id = cur.fetchone()['id']
                    
                    total = Decimal(0)
                    
                    # 2. Agregar items
                    for item in items:
                        # Obtener producto con lock
                        cur.execute(
                            "SELECT precio, stock FROM productos WHERE id = %s FOR UPDATE",
                            (item['producto_id'],)
                        )
                        producto = cur.fetchone()
                        
                        if not producto:
                            raise Exception(f"Producto {item['producto_id']} no existe")
                        
                        if producto['stock'] < item['cantidad']:
                            raise Exception(f"Stock insuficiente para producto {item['producto_id']}")
                        
                        # Insertar item
                        cur.execute(
                            """
                            INSERT INTO items_pedido (pedido_id, producto_id, cantidad, precio)
                            VALUES (%s, %s, %s, %s)
                            """,
                            (pedido_id, item['producto_id'], item['cantidad'], producto['precio'])
                        )
                        
                        # Actualizar stock
                        cur.execute(
                            "UPDATE productos SET stock = stock - %s WHERE id = %s",
                            (item['cantidad'], item['producto_id'])
                        )
                        
                        total += producto['precio'] * item['cantidad']
                    
                    # 3. Actualizar total
                    cur.execute(
                        "UPDATE pedidos SET total = %s WHERE id = %s",
                        (total, pedido_id)
                    )
                    
                    return pedido_id
                    
        except Exception as e:
            print(f"Error al crear pedido: {e}")
            return None
    
    def obtener_completo(self, pedido_id: int) -> Optional[dict]:
        """Obtener pedido con items"""
        with self.db.conn.cursor() as cur:
            # Pedido
            cur.execute(
                """
                SELECT p.*, u.nombre AS cliente, u.email
                FROM pedidos p
                INNER JOIN usuarios u ON p.usuario_id = u.id
                WHERE p.id = %s
                """,
                (pedido_id,)
            )
            pedido = cur.fetchone()
            
            if not pedido:
                return None
            
            # Items
            cur.execute(
                """
                SELECT 
                    i.cantidad,
                    i.precio,
                    i.cantidad * i.precio AS subtotal,
                    prod.nombre AS producto
                FROM items_pedido i
                INNER JOIN productos prod ON i.producto_id = prod.id
                WHERE i.pedido_id = %s
                """,
                (pedido_id,)
            )
            items = cur.fetchall()
            
            pedido['items'] = items
            return pedido

# Uso
def main():
    # Conectar
    with Database("postgresql://app_user:password@localhost/ecommerce") as db:
        # Repositorios
        usuario_repo = UsuarioRepository(db)
        producto_repo = ProductoRepository(db)
        pedido_repo = PedidoRepository(db)
        
        # Crear usuario
        usuario_id = usuario_repo.crear("Ana García", "ana@email.com")
        print(f"Usuario creado: {usuario_id}")
        
        # Crear productos
        producto1_id = producto_repo.crear("Laptop", Decimal("1299.99"), 10, {"marca": "Dell"})
        producto2_id = producto_repo.crear("Mouse", Decimal("25.99"), 50, {"marca": "Logitech"})
        
        # Crear pedido
        pedido_id = pedido_repo.crear(usuario_id, [
            {"producto_id": producto1_id, "cantidad": 1},
            {"producto_id": producto2_id, "cantidad": 2}
        ])
        
        if pedido_id:
            print(f"Pedido creado: {pedido_id}")
            
            # Ver pedido completo
            pedido = pedido_repo.obtener_completo(pedido_id)
            print(f"Cliente: {pedido['cliente']}")
            print(f"Total: ${pedido['total']}")
            print("Items:")
            for item in pedido['items']:
                print(f"  - {item['producto']}: {item['cantidad']} x ${item['precio']} = ${item['subtotal']}")

if __name__ == "__main__":
    main()
```

---

## Resumen

**Instalación:**
```bash
pip install psycopg2-binary
```

**Conexión:**
```python
conn = psycopg2.connect(
    host="localhost",
    database="nombre_bd",
    user="usuario",
    password="password"
)
```

**Queries parametrizadas:**
```python
cur.execute("SELECT * FROM usuarios WHERE email = %s", (email,))
```

**Transacciones:**
```python
try:
    conn.commit()  # Confirmar
except:
    conn.rollback()  # Revertir
```

**Connection Pool:**
```python
pool = pool.SimpleConnectionPool(minconn=1, maxconn=10, ...)
conn = pool.getconn()
```

**Próximos pasos:** SQLAlchemy (ORM), Django ORM

---

## Recursos

- **[psycopg2 Docs](https://www.psycopg.org/docs/)** - Documentación oficial
- **[Python DB-API](https://peps.python.org/pep-0249/)** - Estándar Python para BD
- **[Connection Pool](https://www.psycopg.org/docs/pool.html)** - Pool de conexiones

¡PostgreSQL + Python = 🚀!

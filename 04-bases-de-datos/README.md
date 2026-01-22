# Bases de Datos

> **De conceptos fundamentales a PostgreSQL en producción**

---

## 🎯 Objetivo

Aprender bases de datos desde **cero absoluto**:
- Primero los conceptos (sin código)
- Luego SQL puro (sin frameworks)
- Finalmente PostgreSQL + Python

**Filosofía:** Entender el "por qué" antes del "cómo"

---

## 📚 Contenido

### Bloque 1: Fundamentos (5 módulos)
**Conceptos esenciales sin código**

1. **Introducción a Bases de Datos**
   - ¿Qué es una base de datos?
   - Por qué existen
   - Evolución histórica

2. **Relacionales vs No Relacionales**
   - SQL vs NoSQL
   - Cuándo usar cada una
   - Ventajas y desventajas

3. **Conceptos Fundamentales**
   - Tablas, filas, columnas
   - Claves primarias y foráneas
   - Relaciones (1:1, 1:N, N:M)
   - Integridad referencial

4. **Normalización**
   - Primera forma normal (1NF)
   - Segunda forma normal (2NF)
   - Tercera forma normal (3NF)
   - Desnormalización

5. **Diagramas ERD**
   - Entity-Relationship Diagrams
   - Notaciones (Crow's Foot, Chen, UML)
   - Herramientas de diagramación

---

### Bloque 2: SQL desde cero (8 módulos)
**Lenguaje SQL sin herramientas complejas**

6. **Introducción a SQL**
   - Historia de SQL
   - Tipos de SQL (DDL, DML, DCL, TCL)
   - Sintaxis básica

7. **DDL - Data Definition Language**
   - CREATE TABLE
   - ALTER TABLE
   - DROP TABLE
   - TRUNCATE

8. **DML - Data Manipulation Language**
   - SELECT básico
   - INSERT
   - UPDATE
   - DELETE

9. **Filtrado y Ordenamiento**
   - WHERE (operadores)
   - ORDER BY (ASC/DESC)
   - LIMIT y OFFSET
   - DISTINCT

10. **JOINs**
    - INNER JOIN
    - LEFT JOIN
    - RIGHT JOIN
    - FULL OUTER JOIN
    - CROSS JOIN
    - Self JOIN

11. **Funciones de Agregación**
    - COUNT, SUM, AVG
    - MIN, MAX
    - Funciones de string
    - Funciones de fecha

12. **GROUP BY y HAVING**
    - Agrupaciones
    - HAVING vs WHERE
    - Múltiples agrupaciones

13. **Subconsultas**
    - Subconsultas en WHERE
    - Subconsultas en FROM
    - Subconsultas correlacionadas
    - EXISTS y IN

---

### Bloque 3: PostgreSQL (7 módulos)
**PostgreSQL en profundidad + Python**

14. **Instalación y Configuración**
    - Instalar PostgreSQL
    - Configuración inicial
    - Crear primera base de datos
    - pgAdmin vs psql

15. **psql y Herramientas**
    - Comandos de psql
    - Backup y restore
    - Import/Export CSV
    - Scripts SQL

16. **Tipos de Datos PostgreSQL**
    - Numéricos, strings, fechas
    - JSON y JSONB
    - Arrays
    - UUIDs
    - Tipos personalizados

17. **Índices y Performance**
    - Tipos de índices
    - EXPLAIN y ANALYZE
    - Query optimization
    - Vacuuming

18. **Transacciones y ACID**
    - BEGIN, COMMIT, ROLLBACK
    - Propiedades ACID
    - Niveles de aislamiento
    - Deadlocks

19. **Vistas y Procedimientos**
    - Vistas (VIEW)
    - Vistas materializadas
    - Funciones
    - Procedimientos almacenados
    - Triggers

20. **PostgreSQL con Python**
    - psycopg2
    - Connection pooling
    - Queries parametrizadas
    - ORM vs SQL puro
    - Integración con Django

---

## 🎓 Progresión de aprendizaje

```
Conceptos puros → SQL vanilla → PostgreSQL → Python integration
```

**Módulos 1-5:** Teoría y conceptos (diagramas, papel y lápiz)  
**Módulos 6-13:** SQL puro (SQLite para práctica ligera)  
**Módulos 14-20:** PostgreSQL profesional + Python

---

## 🛠️ Herramientas que usarás

- **SQLite:** Para aprender SQL (sin instalación compleja)
- **PostgreSQL:** Base de datos profesional
- **psql:** Cliente de línea de comandos
- **pgAdmin:** Interfaz gráfica (opcional)
- **psycopg2:** Driver de Python para PostgreSQL
- **DB Browser for SQLite:** GUI para SQLite (opcional)

---

## 📋 Prerequisitos

- ✅ Conocimientos básicos de Python
- ✅ Terminal/línea de comandos
- ✅ Lógica de programación

**No necesitas:**
- ❌ Conocimiento previo de bases de datos
- ❌ Experiencia con SQL
- ❌ Frameworks complejos

---

## 🎯 Después de este módulo sabrás

✅ Diseñar bases de datos relacionales  
✅ Escribir SQL complejo (JOINs, subconsultas)  
✅ Optimizar queries  
✅ Usar PostgreSQL profesionalmente  
✅ Integrar bases de datos con Python  
✅ Entender cuándo usar SQL vs NoSQL

---

## 📖 Recursos adicionales

- **[PostgreSQL Docs](https://www.postgresql.org/docs/)** - Documentación oficial
- **[SQLBolt](https://sqlbolt.com/)** - Tutorial interactivo de SQL
- **[DB Diagram](https://dbdiagram.io/)** - Crear diagramas ERD online
- **[Explain PostgreSQL](https://explain.dalibo.com/)** - Visualizar EXPLAIN

---

**Total:** 20 módulos progresivos

¡Comencemos desde los conceptos! 🚀

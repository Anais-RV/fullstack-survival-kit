# Diagramas ERD (Entity-Relationship Diagrams)

> **Visualizar la estructura de tu base de datos**

---

## ¿Qué es un Diagrama ERD?

**Definición:**  
Un **Diagrama ERD** es una representación visual de las entidades (tablas) y sus relaciones en una base de datos.

**Analogía:**  
Como un plano de una casa muestra habitaciones y cómo conectan.

```
┌─────────────┐         ┌─────────────┐
│  Usuarios   │────────▶│    Posts    │
└─────────────┘   1:N   └─────────────┘
                           │
                           │ 1:N
                           ▼
                      ┌─────────────┐
                      │ Comentarios │
                      └─────────────┘
```

---

## ¿Por qué son importantes?

### ✅ Planificación
Diseñar antes de programar.

### ✅ Comunicación
Explicar estructura al equipo.

### ✅ Documentación
Referencia futura.

### ✅ Detección de errores
Ver problemas antes de implementar.

---

## Componentes de un ERD

### 1. Entidades (Tablas)

Representan **objetos del mundo real**.

```
┌─────────────┐
│  Usuarios   │  ← Entidad
└─────────────┘
```

**Ejemplos:**
- Usuarios
- Productos
- Pedidos
- Clientes

---

### 2. Atributos (Columnas)

**Propiedades** de las entidades.

```
┌─────────────────┐
│    Usuarios     │
├─────────────────┤
│ PK  id          │  ← PRIMARY KEY
│     nombre      │
│     email       │
│     edad        │
└─────────────────┘
```

**Tipos:**
- **PK:** Primary Key (clave primaria)
- **FK:** Foreign Key (clave foránea)
- Atributos normales

---

### 3. Relaciones

**Conexiones** entre entidades.

```
┌──────────┐         ┌──────────┐
│ Usuarios │────────▶│  Posts   │
└──────────┘   1:N   └──────────┘
```

**Tipos:**
- **1:1** (uno a uno)
- **1:N** (uno a muchos)
- **N:M** (muchos a muchos)

---

## Notaciones

Existen varias notaciones para ERD. Las más comunes:

---

## 1. Notación Crow's Foot (Pata de Gallo)

**La más popular.**

### Símbolos

```
─────────  Línea de relación

│          Uno (1)
○          Cero
├──        Uno obligatorio (1)
○──        Cero o uno (0..1)
─<         Muchos (N)
─○<        Cero o muchos (0..N)
─├<        Uno o muchos (1..N)
```

### Relaciones

#### Uno a Uno (1:1)

```
┌──────────┐         ┌──────────┐
│ Usuarios │├───────├│ Perfiles │
└──────────┘         └──────────┘

Un usuario tiene un perfil
Un perfil pertenece a un usuario
```

#### Uno a Muchos (1:N)

```
┌──────────┐         ┌──────────┐
│ Usuarios │├──────<│  Posts   │
└──────────┘         └──────────┘

Un usuario tiene muchos posts
Un post pertenece a un usuario
```

#### Muchos a Muchos (N:M)

```
┌─────────────┐         ┌──────────┐
│ Estudiantes │>──────<│  Cursos  │
└─────────────┘         └──────────┘

Un estudiante toma muchos cursos
Un curso tiene muchos estudiantes
```

---

### Opcionalidad

#### Relación obligatoria (1)

```
┌──────────┐         ┌──────────┐
│  Posts   │├───────├│ Usuarios │
└──────────┘         └──────────┘
             ↑
        obligatorio
(post DEBE tener usuario)
```

#### Relación opcional (0..1)

```
┌──────────┐         ┌───────────┐
│ Usuarios │○───────○│ Direccion │
└──────────┘         └───────────┘
             ↑
        opcional
(usuario PUEDE tener dirección)
```

---

## 2. Notación Chen

**Menos común pero clásica.**

```
┌─────────┐                    ┌─────────┐
│Usuarios │───── tiene ────────│  Posts  │
└─────────┘       1    N       └─────────┘
     │                              │
     │                              │
  nombre                         titulo
   email                       contenido
```

**Características:**
- Relaciones en rombos
- Cardinalidad con números

---

## 3. Notación UML

**Para diagramas orientados a objetos.**

```
┌────────────────────┐
│      Usuarios      │
├────────────────────┤
│ - id: int          │
│ - nombre: string   │
│ - email: string    │
├────────────────────┤
│ + crear()          │
│ + actualizar()     │
└────────────────────┘
```

---

## Ejemplo Completo: Blog

### Diagrama Crow's Foot

```
┌──────────────────┐
│     Usuarios     │
├──────────────────┤
│ PK  id           │
│     nombre       │
│     email        │
│     password     │
└──────────────────┘
         │
         │ 1:N
         ▼
┌──────────────────┐
│      Posts       │
├──────────────────┤
│ PK  id           │
│ FK  usuario_id   │
│     titulo       │
│     contenido    │
│     fecha        │
└──────────────────┘
         │
         │ 1:N
         ▼
┌──────────────────┐
│   Comentarios    │
├──────────────────┤
│ PK  id           │
│ FK  post_id      │
│ FK  usuario_id   │
│     texto        │
│     fecha        │
└──────────────────┘
         ▲
         │ N:1
         │
┌──────────────────┐
│     Usuarios     │
└──────────────────┘
```

### SQL Equivalente

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(255) UNIQUE,
    password VARCHAR(255)
);

CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    usuario_id INTEGER NOT NULL,
    titulo VARCHAR(200),
    contenido TEXT,
    fecha TIMESTAMP,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);

CREATE TABLE comentarios (
    id INTEGER PRIMARY KEY,
    post_id INTEGER NOT NULL,
    usuario_id INTEGER NOT NULL,
    texto TEXT,
    fecha TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(id),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

---

## Ejemplo: E-commerce

### Diagrama

```
┌──────────────────┐
│     Clientes     │
├──────────────────┤
│ PK  id           │
│     nombre       │
│     email        │
│     telefono     │
└──────────────────┘
         │
         │ 1:N
         ▼
┌──────────────────┐
│     Pedidos      │
├──────────────────┤
│ PK  id           │
│ FK  cliente_id   │
│     fecha        │
│     total        │
│     estado       │
└──────────────────┘
         │
         │ 1:N
         ▼
┌──────────────────┐              ┌──────────────────┐
│  Items_Pedido    │     N:1      │    Productos     │
├──────────────────┤────────────▶├──────────────────┤
│ PK  id           │              │ PK  id           │
│ FK  pedido_id    │              │     nombre       │
│ FK  producto_id  │              │     descripcion  │
│     cantidad     │              │     precio       │
│     precio_unit  │              │     stock        │
└──────────────────┘              └──────────────────┘
                                           │
                                           │ N:1
                                           ▼
                                  ┌──────────────────┐
                                  │   Categorias     │
                                  ├──────────────────┤
                                  │ PK  id           │
                                  │     nombre       │
                                  │     descripcion  │
                                  └──────────────────┘
```

### Relaciones

**1. Clientes → Pedidos (1:N)**
- Un cliente puede tener muchos pedidos
- Un pedido pertenece a un cliente

**2. Pedidos → Items_Pedido (1:N)**
- Un pedido tiene muchos items
- Un item pertenece a un pedido

**3. Productos → Items_Pedido (1:N)**
- Un producto puede estar en muchos items
- Un item se refiere a un producto

**4. Categorias → Productos (1:N)**
- Una categoría tiene muchos productos
- Un producto pertenece a una categoría

---

## Ejemplo: Relación N:M

### Estudiantes ↔ Cursos

```
┌──────────────────┐                         ┌──────────────────┐
│   Estudiantes    │                         │      Cursos      │
├──────────────────┤                         ├──────────────────┤
│ PK  id           │                         │ PK  id           │
│     nombre       │                         │     nombre       │
│     email        │                         │     creditos     │
│     matricula    │                         │     descripcion  │
└──────────────────┘                         └──────────────────┘
         │                                            │
         │ N                                        M │
         └──────────────┬──────────────────────────┘
                        │
                        ▼
               ┌──────────────────┐
               │  Inscripciones   │  ← Tabla intermedia
               ├──────────────────┤
               │ FK  estudiante_id│
               │ FK  curso_id     │
               │     fecha_insc   │
               │     calificacion │
               └──────────────────┘
                  PRIMARY KEY
                (estudiante_id, curso_id)
```

### SQL

```sql
CREATE TABLE estudiantes (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(255),
    matricula VARCHAR(20) UNIQUE
);

CREATE TABLE cursos (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(200),
    creditos INTEGER,
    descripcion TEXT
);

CREATE TABLE inscripciones (
    estudiante_id INTEGER,
    curso_id INTEGER,
    fecha_inscripcion DATE,
    calificacion DECIMAL(3,1),
    PRIMARY KEY (estudiante_id, curso_id),
    FOREIGN KEY (estudiante_id) REFERENCES estudiantes(id),
    FOREIGN KEY (curso_id) REFERENCES cursos(id)
);
```

---

## Cardinalidad y Modalidad

### Cardinalidad

**¿Cuántos?**

- **1:** Uno
- **N:** Muchos
- **M:** Muchos (otra cantidad)

### Modalidad

**¿Es obligatorio?**

- **Obligatorio (├):** Debe existir
- **Opcional (○):** Puede no existir

### Ejemplos

#### Post → Usuario (obligatorio)

```
┌──────────┐         ┌──────────┐
│  Posts   │├───────├│ Usuarios │
└──────────┘         └──────────┘

Post DEBE tener usuario
Usuario PUEDE no tener posts
```

#### Usuario → Dirección (opcional)

```
┌──────────┐         ┌───────────┐
│ Usuarios │○───────○│ Direccion │
└──────────┘         └───────────┘

Usuario PUEDE tener dirección
Dirección PUEDE no tener usuario
```

---

## Herramientas para Crear ERD

### 1. dbdiagram.io

**Online, gratuito**

```
// Sintaxis simple
Table usuarios {
  id integer [primary key]
  nombre varchar
  email varchar [unique]
}

Table posts {
  id integer [primary key]
  usuario_id integer [ref: > usuarios.id]
  titulo varchar
}
```

**URL:** https://dbdiagram.io

---

### 2. draw.io (diagrams.net)

**Gratis, offline**

- Arrastrar y soltar
- Múltiples plantillas
- Exportar a PNG, PDF

**URL:** https://app.diagrams.net

---

### 3. Lucidchart

**Profesional, pago**

- Colaboración en tiempo real
- Plantillas avanzadas

**URL:** https://www.lucidchart.com

---

### 4. MySQL Workbench

**Gratis, específico para MySQL**

- Reverse engineering (generar ERD desde BD existente)
- Forward engineering (generar SQL desde ERD)

---

### 5. pgAdmin / DBeaver

**Gratis, para PostgreSQL y otras BD**

- Visualizar estructura existente

---

## Proceso de Diseño

### 1. Identificar entidades

**Pregunta:** ¿Qué cosas necesito almacenar?

**Ejemplo: Red social**
- Usuarios
- Posts
- Comentarios
- Likes

---

### 2. Definir atributos

**Pregunta:** ¿Qué información necesito de cada entidad?

**Usuarios:**
- id (PK)
- nombre
- email
- password
- fecha_registro

**Posts:**
- id (PK)
- usuario_id (FK)
- contenido
- fecha_creacion

---

### 3. Identificar relaciones

**Pregunta:** ¿Cómo se conectan?

- Usuarios → Posts (1:N)
- Posts → Comentarios (1:N)
- Usuarios → Comentarios (1:N)
- Usuarios ↔ Posts (N:M para likes)

---

### 4. Definir cardinalidad

**Pregunta:** ¿Cuántos pueden relacionarse?

- Un usuario → muchos posts (1:N)
- Un post → un usuario (N:1)

---

### 5. Verificar normalización

**Pregunta:** ¿Hay duplicación? ¿Dependencias transitivas?

---

### 6. Crear diagrama

**Herramienta:** dbdiagram.io, draw.io, etc.

---

## Ejemplo Completo: Biblioteca

### Requisitos

- **Libros** (título, ISBN, año)
- **Autores** (nombre, nacionalidad)
- **Usuarios** (nombre, email)
- **Préstamos** (fecha inicio, fecha fin)
- Un libro puede tener varios autores
- Un autor puede escribir varios libros
- Un usuario puede tener varios préstamos
- Un libro puede ser prestado varias veces

---

### Diagrama ERD

```
┌──────────────────┐                         ┌──────────────────┐
│     Autores      │                         │      Libros      │
├──────────────────┤                         ├──────────────────┤
│ PK  id           │                         │ PK  id           │
│     nombre       │                         │     titulo       │
│     nacionalidad │                         │     isbn         │
│     fecha_nac    │                         │     anio         │
└──────────────────┘                         └──────────────────┘
         │                                            │
         │ N                                        M │
         └──────────────┬──────────────────────────┘
                        │
                        ▼
               ┌──────────────────┐
               │ Libros_Autores   │  ← N:M
               ├──────────────────┤
               │ FK  libro_id     │
               │ FK  autor_id     │
               │     orden        │  (primer autor, segundo autor)
               └──────────────────┘


┌──────────────────┐
│     Usuarios     │
├──────────────────┤
│ PK  id           │
│     nombre       │
│     email        │
│     telefono     │
└──────────────────┘
         │
         │ 1:N
         ▼
┌──────────────────┐
│    Prestamos     │
├──────────────────┤
│ PK  id           │
│ FK  libro_id     │
│ FK  usuario_id   │
│     fecha_inicio │
│     fecha_fin    │
│     devuelto     │
└──────────────────┘
         │
         │ N:1
         ▼
┌──────────────────┐
│      Libros      │
└──────────────────┘
```

---

### SQL

```sql
CREATE TABLE autores (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(200),
    nacionalidad VARCHAR(100),
    fecha_nacimiento DATE
);

CREATE TABLE libros (
    id INTEGER PRIMARY KEY,
    titulo VARCHAR(300),
    isbn VARCHAR(13) UNIQUE,
    anio INTEGER
);

CREATE TABLE libros_autores (
    libro_id INTEGER,
    autor_id INTEGER,
    orden INTEGER,  -- primer autor, segundo autor
    PRIMARY KEY (libro_id, autor_id),
    FOREIGN KEY (libro_id) REFERENCES libros(id),
    FOREIGN KEY (autor_id) REFERENCES autores(id)
);

CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(255) UNIQUE,
    telefono VARCHAR(20)
);

CREATE TABLE prestamos (
    id INTEGER PRIMARY KEY,
    libro_id INTEGER NOT NULL,
    usuario_id INTEGER NOT NULL,
    fecha_inicio DATE NOT NULL,
    fecha_fin DATE,
    devuelto BOOLEAN DEFAULT false,
    FOREIGN KEY (libro_id) REFERENCES libros(id),
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

---

## Buenas Prácticas

### ✅ Nombres claros

```
Usuarios (no: User, usr, u)
Productos (no: prod, p)
```

### ✅ Singular o plural consistente

```
Opción 1: usuario, post, comentario
Opción 2: usuarios, posts, comentarios

Elige una y mantén consistencia
```

### ✅ Siempre id como PRIMARY KEY

```sql
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,  -- estándar
    ...
);
```

### ✅ Foreign keys con sufijo _id

```sql
CREATE TABLE posts (
    usuario_id INTEGER,  -- claramente es FK
    ...
);
```

### ✅ Evitar abreviaciones

```
email (no: eml, e)
nombre (no: nom, n)
```

---

## Errores Comunes

### ❌ No modelar relaciones N:M

```
-- MAL: guardar IDs separados por coma
CREATE TABLE estudiantes (
    cursos VARCHAR(255)  -- "1,2,3" ← ❌
);

-- BIEN: tabla intermedia
CREATE TABLE inscripciones (
    estudiante_id INTEGER,
    curso_id INTEGER
);
```

### ❌ Datos en nombres de columnas

```
-- MAL
CREATE TABLE ventas (
    producto_1 VARCHAR(100),
    producto_2 VARCHAR(100),
    producto_3 VARCHAR(100)
);

-- BIEN
CREATE TABLE items_venta (
    venta_id INTEGER,
    producto VARCHAR(100)
);
```

### ❌ Duplicar información

```
-- MAL
CREATE TABLE pedidos (
    cliente_nombre VARCHAR(100),
    cliente_email VARCHAR(255)  -- duplicado en cada pedido
);

-- BIEN
CREATE TABLE pedidos (
    cliente_id INTEGER  -- referencia a tabla clientes
);
```

---

## Resumen

**Diagrama ERD:**
- 📊 Visualiza estructura de BD
- 🔗 Muestra entidades y relaciones
- 📐 Planificar antes de implementar

**Componentes:**
- **Entidades:** Tablas
- **Atributos:** Columnas (PK, FK)
- **Relaciones:** 1:1, 1:N, N:M

**Notaciones:**
- **Crow's Foot:** Más popular
- **Chen:** Clásica
- **UML:** Orientada a objetos

**Herramientas:**
- dbdiagram.io (online, gratis)
- draw.io (offline, gratis)
- MySQL Workbench (MySQL)

**Proceso:**
1. Identificar entidades
2. Definir atributos
3. Identificar relaciones
4. Definir cardinalidad
5. Verificar normalización
6. Crear diagrama

**Próximo módulo:** SQL desde cero (empezar a escribir consultas)

---

## Recursos

- **[dbdiagram.io](https://dbdiagram.io)** - Crear ERDs online
- **[draw.io](https://app.diagrams.net)** - Diagramas gratis
- **[ERD Tutorial](https://www.lucidchart.com/pages/er-diagrams)** - Guía completa
- **[Crow's Foot Notation](https://www.freecodecamp.org/news/crows-foot-notation-relationship-symbols-and-how-to-read-diagrams/)** - Explicación detallada

Ahora puedes **diseñar y visualizar** bases de datos. 📐

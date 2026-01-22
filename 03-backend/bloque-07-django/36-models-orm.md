# Django Models y ORM

> **Interactúa con bases de datos sin escribir SQL**

---

## ¿Qué es el ORM de Django?

**ORM** = Object-Relational Mapping

**Convierte:**
```python
# Python
Usuario.objects.filter(activo=True)

# ↓ Django genera automáticamente ↓

# SQL
SELECT * FROM usuarios WHERE activo = TRUE;
```

---

## Recordatorio: Vanilla vs Django

**Vanilla (lo que hiciste):**
```python
import sqlite3

conn = sqlite3.connect('db.sqlite3')
cursor = conn.cursor()

cursor.execute('''
    CREATE TABLE usuarios (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nombre TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        activo BOOLEAN DEFAULT 1
    )
''')

cursor.execute('''
    INSERT INTO usuarios (nombre, email) VALUES (?, ?)
''', ('Ana', 'ana@example.com'))

conn.commit()
```

**Django:**
```python
class Usuario(models.Model):
    nombre = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    activo = models.BooleanField(default=True)

# Uso
Usuario.objects.create(nombre='Ana', email='ana@example.com')
```

---

## Definir modelo básico

**models.py**

```python
from django.db import models

class Usuario(models.Model):
    """Modelo de usuario"""
    nombre = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    edad = models.IntegerField(null=True, blank=True)
    activo = models.BooleanField(default=True)
    creado_en = models.DateTimeField(auto_now_add=True)
    actualizado_en = models.DateTimeField(auto_now=True)
    
    class Meta:
        db_table = 'usuarios'
        ordering = ['-creado_en']
        verbose_name = 'Usuario'
        verbose_name_plural = 'Usuarios'
    
    def __str__(self):
        return f'{self.nombre} ({self.email})'
    
    def es_adulto(self):
        """Método personalizado"""
        return self.edad and self.edad >= 18
```

---

## Tipos de campos

### CharField

```python
class Producto(models.Model):
    nombre = models.CharField(
        max_length=200,
        help_text='Nombre del producto'
    )
    codigo = models.CharField(
        max_length=50,
        unique=True,
        db_index=True  # Índice en BD
    )
```

### TextField

```python
class Articulo(models.Model):
    titulo = models.CharField(max_length=200)
    contenido = models.TextField()  # Texto largo sin límite
    resumen = models.TextField(blank=True, default='')
```

### EmailField

```python
class Usuario(models.Model):
    email = models.EmailField(unique=True)
    email_alternativo = models.EmailField(blank=True, null=True)
```

### IntegerField

```python
class Producto(models.Model):
    stock = models.IntegerField(default=0)
    precio = models.IntegerField()  # Centavos (para evitar decimales)
    
    @property
    def precio_dolares(self):
        return self.precio / 100
```

### DecimalField

```python
class Producto(models.Model):
    precio = models.DecimalField(
        max_digits=10,  # Total de dígitos
        decimal_places=2  # Decimales
    )
    # Ej: 12345678.90 (8 enteros + 2 decimales)
```

### BooleanField

```python
class Usuario(models.Model):
    activo = models.BooleanField(default=True)
    verificado = models.BooleanField(default=False)
```

### DateTimeField

```python
class Post(models.Model):
    creado_en = models.DateTimeField(auto_now_add=True)  # Solo al crear
    actualizado_en = models.DateTimeField(auto_now=True)  # Cada save()
    publicado_en = models.DateTimeField(null=True, blank=True)
```

### DateField

```python
class Empleado(models.Model):
    fecha_nacimiento = models.DateField()
    fecha_ingreso = models.DateField(auto_now_add=True)
```

### TimeField

```python
class Horario(models.Model):
    hora_apertura = models.TimeField()
    hora_cierre = models.TimeField()
```

### JSONField

```python
class Configuracion(models.Model):
    datos = models.JSONField(default=dict)
    
# Uso
config = Configuracion.objects.create(
    datos={
        'tema': 'oscuro',
        'idioma': 'es',
        'notificaciones': {
            'email': True,
            'push': False
        }
    }
)
```

---

## Opciones de campos

### null y blank

```python
class Persona(models.Model):
    # null=True: Permite NULL en BD
    # blank=True: Permite vacío en formularios
    
    telefono = models.CharField(
        max_length=20,
        null=True,  # BD puede ser NULL
        blank=True  # Formulario puede estar vacío
    )
    
    # Solo blank (para strings)
    biografia = models.TextField(
        blank=True,
        default=''  # Mejor que null para strings
    )
```

**Regla general:**
- **Strings**: `blank=True, default=''` (no usar `null=True`)
- **Otros tipos**: `null=True, blank=True`

### default

```python
class Producto(models.Model):
    stock = models.IntegerField(default=0)
    activo = models.BooleanField(default=True)
    tags = models.JSONField(default=list)  # ¡Usar callable!
```

**⚠️ Cuidado con defaults mutables:**

```python
# ❌ INCORRECTO
tags = models.JSONField(default=[])  # Compartirán la misma lista

# ✅ CORRECTO
tags = models.JSONField(default=list)  # Nueva lista cada vez
```

### unique

```python
class Usuario(models.Model):
    email = models.EmailField(unique=True)
    username = models.CharField(max_length=50, unique=True)
```

### choices

```python
class Pedido(models.Model):
    ESTADO_CHOICES = [
        ('pendiente', 'Pendiente'),
        ('pagado', 'Pagado'),
        ('enviado', 'Enviado'),
        ('entregado', 'Entregado'),
        ('cancelado', 'Cancelado'),
    ]
    
    estado = models.CharField(
        max_length=20,
        choices=ESTADO_CHOICES,
        default='pendiente'
    )

# Uso
pedido = Pedido.objects.create(estado='pagado')
print(pedido.get_estado_display())  # "Pagado"
```

**Mejor con Enums (Django 3.0+):**

```python
from django.db import models

class Pedido(models.Model):
    class Estado(models.TextChoices):
        PENDIENTE = 'pendiente', 'Pendiente'
        PAGADO = 'pagado', 'Pagado'
        ENVIADO = 'enviado', 'Enviado'
        ENTREGADO = 'entregado', 'Entregado'
        CANCELADO = 'cancelado', 'Cancelado'
    
    estado = models.CharField(
        max_length=20,
        choices=Estado.choices,
        default=Estado.PENDIENTE
    )

# Uso
pedido.estado = Pedido.Estado.PAGADO
print(pedido.get_estado_display())  # "Pagado"
```

### db_index

```python
class Usuario(models.Model):
    email = models.EmailField(
        unique=True,  # Ya crea índice
        db_index=True  # No necesario si unique=True
    )
    
    nombre = models.CharField(
        max_length=100,
        db_index=True  # Búsquedas frecuentes por nombre
    )
```

---

## Relaciones entre modelos

### ForeignKey (Muchos a Uno)

```python
class Autor(models.Model):
    nombre = models.CharField(max_length=100)
    
    def __str__(self):
        return self.nombre

class Libro(models.Model):
    titulo = models.CharField(max_length=200)
    autor = models.ForeignKey(
        Autor,
        on_delete=models.CASCADE,  # Si se borra autor, borrar libros
        related_name='libros'  # autor.libros.all()
    )
    
    def __str__(self):
        return self.titulo
```

**Opciones de on_delete:**

```python
# CASCADE: Borrar en cascada
autor = models.ForeignKey(Autor, on_delete=models.CASCADE)

# PROTECT: No permitir borrar si hay relaciones
autor = models.ForeignKey(Autor, on_delete=models.PROTECT)

# SET_NULL: Poner en NULL (requiere null=True)
autor = models.ForeignKey(Autor, on_delete=models.SET_NULL, null=True)

# SET_DEFAULT: Poner valor por defecto
autor = models.ForeignKey(Autor, on_delete=models.SET_DEFAULT, default=1)

# SET: Poner valor específico
def get_autor_default():
    return Autor.objects.get_or_create(nombre='Desconocido')[0]

autor = models.ForeignKey(Autor, on_delete=models.SET(get_autor_default))

# DO_NOTHING: No hacer nada (puede romper BD)
autor = models.ForeignKey(Autor, on_delete=models.DO_NOTHING)
```

**Uso:**

```python
# Crear
autor = Autor.objects.create(nombre='García Márquez')
libro = Libro.objects.create(titulo='Cien años de soledad', autor=autor)

# Acceder
print(libro.autor.nombre)  # "García Márquez"
print(autor.libros.all())  # QuerySet de libros del autor
```

### OneToOneField (Uno a Uno)

```python
class Usuario(models.Model):
    username = models.CharField(max_length=50, unique=True)
    email = models.EmailField()

class Perfil(models.Model):
    usuario = models.OneToOneField(
        Usuario,
        on_delete=models.CASCADE,
        related_name='perfil'
    )
    biografia = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    fecha_nacimiento = models.DateField(null=True, blank=True)
```

**Uso:**

```python
# Crear
usuario = Usuario.objects.create(username='ana', email='ana@example.com')
perfil = Perfil.objects.create(usuario=usuario, biografia='Desarrolladora')

# Acceder
print(usuario.perfil.biografia)  # "Desarrolladora"
print(perfil.usuario.username)  # "ana"
```

### ManyToManyField (Muchos a Muchos)

```python
class Estudiante(models.Model):
    nombre = models.CharField(max_length=100)
    
    def __str__(self):
        return self.nombre

class Curso(models.Model):
    nombre = models.CharField(max_length=100)
    estudiantes = models.ManyToManyField(
        Estudiante,
        related_name='cursos'
    )
    
    def __str__(self):
        return self.nombre
```

**Uso:**

```python
# Crear
estudiante1 = Estudiante.objects.create(nombre='Ana')
estudiante2 = Estudiante.objects.create(nombre='Bob')

curso = Curso.objects.create(nombre='Python')

# Agregar relaciones
curso.estudiantes.add(estudiante1)
curso.estudiantes.add(estudiante2)

# O múltiples
curso.estudiantes.add(estudiante1, estudiante2)

# O con set
curso.estudiantes.set([estudiante1, estudiante2])

# Acceder
print(curso.estudiantes.all())  # Ana, Bob
print(estudiante1.cursos.all())  # Python

# Remover
curso.estudiantes.remove(estudiante1)

# Limpiar todos
curso.estudiantes.clear()
```

### ManyToMany con tabla intermedia

```python
class Estudiante(models.Model):
    nombre = models.CharField(max_length=100)

class Curso(models.Model):
    nombre = models.CharField(max_length=100)
    estudiantes = models.ManyToManyField(
        Estudiante,
        through='Inscripcion',
        related_name='cursos'
    )

class Inscripcion(models.Model):
    """Tabla intermedia con campos adicionales"""
    estudiante = models.ForeignKey(Estudiante, on_delete=models.CASCADE)
    curso = models.ForeignKey(Curso, on_delete=models.CASCADE)
    fecha_inscripcion = models.DateField(auto_now_add=True)
    calificacion = models.DecimalField(
        max_digits=4,
        decimal_places=2,
        null=True,
        blank=True
    )
    
    class Meta:
        unique_together = ['estudiante', 'curso']
```

**Uso:**

```python
# Crear con tabla intermedia
estudiante = Estudiante.objects.create(nombre='Ana')
curso = Curso.objects.create(nombre='Python')

inscripcion = Inscripcion.objects.create(
    estudiante=estudiante,
    curso=curso,
    calificacion=9.5
)

# Acceder
print(estudiante.cursos.all())
print(curso.estudiantes.all())

# Acceder a datos de la tabla intermedia
inscripciones = Inscripcion.objects.filter(estudiante=estudiante)
for ins in inscripciones:
    print(f'{ins.curso.nombre}: {ins.calificacion}')
```

---

## Meta opciones

```python
class Libro(models.Model):
    titulo = models.CharField(max_length=200)
    autor = models.ForeignKey(Autor, on_delete=models.CASCADE)
    publicado_en = models.DateField()
    isbn = models.CharField(max_length=13, unique=True)
    
    class Meta:
        # Nombre de tabla en BD
        db_table = 'libros'
        
        # Orden por defecto
        ordering = ['-publicado_en', 'titulo']
        
        # Nombres legibles
        verbose_name = 'Libro'
        verbose_name_plural = 'Libros'
        
        # Índices compuestos
        indexes = [
            models.Index(fields=['autor', 'publicado_en']),
        ]
        
        # Restricciones únicas compuestas
        unique_together = [['autor', 'titulo']]
        
        # Permisos personalizados
        permissions = [
            ('puede_publicar', 'Puede publicar libros'),
        ]
        
        # App específica (si se mueve)
        app_label = 'biblioteca'
```

---

## Métodos del modelo

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=200)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    activo = models.BooleanField(default=True)
    
    def __str__(self):
        """Representación en string"""
        return self.nombre
    
    def save(self, *args, **kwargs):
        """Override save para lógica personalizada"""
        # Desactivar si no hay stock
        if self.stock == 0:
            self.activo = False
        
        super().save(*args, **kwargs)
    
    def delete(self, *args, **kwargs):
        """Override delete"""
        print(f'Eliminando producto: {self.nombre}')
        super().delete(*args, **kwargs)
    
    @property
    def precio_con_iva(self):
        """Property calculada"""
        return self.precio * 1.21
    
    def tiene_stock(self):
        """Método personalizado"""
        return self.stock > 0
    
    def reducir_stock(self, cantidad):
        """Método de negocio"""
        if self.stock < cantidad:
            raise ValueError('Stock insuficiente')
        
        self.stock -= cantidad
        self.save()
    
    def get_absolute_url(self):
        """URL del objeto"""
        from django.urls import reverse
        return reverse('producto_detail', kwargs={'pk': self.pk})
```

---

## QuerySets básicos

### Crear objetos

```python
# Método 1: create()
usuario = Usuario.objects.create(
    nombre='Ana',
    email='ana@example.com'
)

# Método 2: save()
usuario = Usuario(nombre='Bob', email='bob@example.com')
usuario.save()

# Método 3: get_or_create()
usuario, created = Usuario.objects.get_or_create(
    email='ana@example.com',
    defaults={'nombre': 'Ana García'}
)

if created:
    print('Usuario creado')
else:
    print('Usuario ya existía')
```

### Obtener objetos

```python
# get(): Un solo objeto (error si 0 o >1)
usuario = Usuario.objects.get(id=1)
usuario = Usuario.objects.get(email='ana@example.com')

# all(): Todos los objetos
usuarios = Usuario.objects.all()

# filter(): Filtrar (retorna QuerySet)
usuarios = Usuario.objects.filter(activo=True)

# exclude(): Excluir
usuarios = Usuario.objects.exclude(activo=False)

# first() / last()
primer_usuario = Usuario.objects.first()
ultimo_usuario = Usuario.objects.last()

# exists()
hay_usuarios = Usuario.objects.exists()  # True/False

# count()
total = Usuario.objects.count()
```

### Actualizar objetos

```python
# Método 1: Modificar y save()
usuario = Usuario.objects.get(id=1)
usuario.nombre = 'Ana García'
usuario.save()

# Método 2: update() (más eficiente para múltiples)
Usuario.objects.filter(activo=False).update(activo=True)

# update_or_create()
usuario, created = Usuario.objects.update_or_create(
    email='ana@example.com',
    defaults={'nombre': 'Ana María'}
)
```

### Eliminar objetos

```python
# Eliminar uno
usuario = Usuario.objects.get(id=1)
usuario.delete()

# Eliminar múltiples
Usuario.objects.filter(activo=False).delete()

# Eliminar todos (¡cuidado!)
Usuario.objects.all().delete()
```

---

## Lookups (búsquedas avanzadas)

```python
# Exacto (por defecto)
Usuario.objects.filter(nombre='Ana')
Usuario.objects.filter(nombre__exact='Ana')

# Case-insensitive
Usuario.objects.filter(nombre__iexact='ana')

# Contiene
Usuario.objects.filter(nombre__contains='ar')  # "García", "Mario"
Usuario.objects.filter(nombre__icontains='AR')  # Case-insensitive

# Empieza/termina con
Usuario.objects.filter(email__startswith='ana')
Usuario.objects.filter(email__endswith='@gmail.com')

# Mayor/menor que
Producto.objects.filter(precio__gt=100)  # >
Producto.objects.filter(precio__gte=100)  # >=
Producto.objects.filter(precio__lt=50)  # <
Producto.objects.filter(precio__lte=50)  # <=

# Rango
Producto.objects.filter(precio__range=(10, 100))

# En lista
Usuario.objects.filter(id__in=[1, 2, 3])

# NULL
Usuario.objects.filter(telefono__isnull=True)
Usuario.objects.filter(telefono__isnull=False)

# Fechas
from datetime import date, timedelta

hoy = date.today()
ayer = hoy - timedelta(days=1)

Post.objects.filter(creado_en__date=hoy)
Post.objects.filter(creado_en__year=2024)
Post.objects.filter(creado_en__month=12)
Post.objects.filter(creado_en__day=25)
Post.objects.filter(creado_en__gte=ayer)

# Regex
Usuario.objects.filter(email__regex=r'^[a-z]+@gmail\.com$')
Usuario.objects.filter(email__iregex=r'^[A-Z]+@GMAIL\.COM$')
```

---

## Combinando filtros

```python
# AND implícito
Usuario.objects.filter(activo=True, edad__gte=18)

# AND explícito con Q
from django.db.models import Q

Usuario.objects.filter(Q(activo=True) & Q(edad__gte=18))

# OR
Usuario.objects.filter(Q(edad__lt=18) | Q(edad__gt=65))

# NOT
Usuario.objects.filter(~Q(activo=True))

# Combinaciones complejas
Usuario.objects.filter(
    (Q(edad__lt=18) | Q(edad__gt=65)) & Q(activo=True)
)

# Múltiples filter (AND)
Usuario.objects.filter(activo=True).filter(edad__gte=18)
```

---

## Relaciones en queries

```python
# ForeignKey: Acceso directo
libro = Libro.objects.get(id=1)
print(libro.autor.nombre)

# Reverse: related_name
autor = Autor.objects.get(id=1)
print(autor.libros.all())

# Filtrar por relación (lookup con __)
libros = Libro.objects.filter(autor__nombre='García Márquez')
libros = Libro.objects.filter(autor__nombre__contains='García')

# Reverse filter
autores = Autor.objects.filter(libros__titulo__contains='Python')

# ManyToMany
curso = Curso.objects.get(id=1)
estudiantes = curso.estudiantes.all()

estudiante = Estudiante.objects.get(id=1)
cursos = estudiante.cursos.all()

# Filtrar por M2M
estudiantes = Estudiante.objects.filter(cursos__nombre='Python')
cursos = Curso.objects.filter(estudiantes__nombre='Ana')
```

---

## select_related (JOIN)

**Para ForeignKey y OneToOne**

```python
# ❌ Sin select_related (N+1 queries)
libros = Libro.objects.all()
for libro in libros:
    print(libro.autor.nombre)  # Query por cada libro

# ✅ Con select_related (1 query con JOIN)
libros = Libro.objects.select_related('autor').all()
for libro in libros:
    print(libro.autor.nombre)  # Sin queries adicionales
```

**SQL generado:**

```sql
SELECT libro.*, autor.*
FROM libro
INNER JOIN autor ON libro.autor_id = autor.id
```

---

## prefetch_related (Queries separadas)

**Para ManyToMany y Reverse ForeignKey**

```python
# ❌ Sin prefetch_related (N+1 queries)
cursos = Curso.objects.all()
for curso in cursos:
    print(curso.estudiantes.all())  # Query por cada curso

# ✅ Con prefetch_related (2 queries totales)
cursos = Curso.objects.prefetch_related('estudiantes').all()
for curso in cursos:
    print(curso.estudiantes.all())  # Sin queries adicionales
```

**SQL generado:**

```sql
-- Query 1
SELECT * FROM curso

-- Query 2
SELECT * FROM estudiante
INNER JOIN curso_estudiante ON estudiante.id = curso_estudiante.estudiante_id
WHERE curso_estudiante.curso_id IN (1, 2, 3, ...)
```

**Combinados:**

```python
# ForeignKey + ManyToMany
libros = Libro.objects.select_related('autor').prefetch_related('categorias')
```

---

## Agregaciones

```python
from django.db.models import Count, Avg, Sum, Min, Max

# Count
total_usuarios = Usuario.objects.count()

# Agregaciones
stats = Producto.objects.aggregate(
    total=Count('id'),
    precio_promedio=Avg('precio'),
    precio_max=Max('precio'),
    precio_min=Min('precio'),
    valor_inventario=Sum('precio')
)

# Resultado:
# {
#     'total': 100,
#     'precio_promedio': 45.50,
#     'precio_max': 200.00,
#     'precio_min': 5.00,
#     'valor_inventario': 4550.00
# }
```

---

## Anotaciones (annotate)

```python
from django.db.models import Count, Sum, F

# Contar libros por autor
autores = Autor.objects.annotate(
    total_libros=Count('libros')
)

for autor in autores:
    print(f'{autor.nombre}: {autor.total_libros} libros')

# Sumar stock por categoría
categorias = Categoria.objects.annotate(
    stock_total=Sum('productos__stock')
)

# Campos calculados con F()
productos = Producto.objects.annotate(
    valor_stock=F('precio') * F('stock')
)
```

---

## Ordenamiento

```python
# Ascendente
Usuario.objects.order_by('nombre')

# Descendente
Usuario.objects.order_by('-creado_en')

# Múltiples campos
Usuario.objects.order_by('activo', '-creado_en')

# Aleatorio
Usuario.objects.order_by('?')

# Inverso
Usuario.objects.order_by('-nombre').reverse()
```

---

## Limitar resultados

```python
# Primeros 10
Usuario.objects.all()[:10]

# Del 10 al 20
Usuario.objects.all()[10:20]

# Primero
Usuario.objects.first()

# Último
Usuario.objects.last()
```

---

## Valores específicos

```python
# values(): Diccionarios
usuarios = Usuario.objects.values('id', 'nombre', 'email')
# [{'id': 1, 'nombre': 'Ana', 'email': 'ana@example.com'}, ...]

# values_list(): Tuplas
usuarios = Usuario.objects.values_list('nombre', 'email')
# [('Ana', 'ana@example.com'), ...]

# values_list con flat=True (un solo campo)
nombres = Usuario.objects.values_list('nombre', flat=True)
# ['Ana', 'Bob', 'Carlos']
```

---

## Queries raw

```python
# Raw SQL
usuarios = Usuario.objects.raw('SELECT * FROM usuarios WHERE edad > %s', [18])

# Cursor directo
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute('SELECT * FROM usuarios WHERE edad > %s', [18])
    rows = cursor.fetchall()
```

---

## Transacciones

```python
from django.db import transaction

# Decorador
@transaction.atomic
def transferir_dinero(origen, destino, monto):
    origen.saldo -= monto
    origen.save()
    
    destino.saldo += monto
    destino.save()

# Context manager
from django.db import transaction

def procesar_pedido(pedido):
    try:
        with transaction.atomic():
            # Reducir stock
            for item in pedido.items.all():
                producto = item.producto
                producto.stock -= item.cantidad
                producto.save()
            
            # Confirmar pedido
            pedido.estado = 'confirmado'
            pedido.save()
            
    except Exception as e:
        # Rollback automático
        print(f'Error: {e}')
```

---

## Ejemplo completo: E-commerce

```python
from django.db import models
from django.contrib.auth.models import User

class Categoria(models.Model):
    nombre = models.CharField(max_length=100)
    slug = models.SlugField(unique=True)
    
    class Meta:
        verbose_name_plural = 'Categorías'
    
    def __str__(self):
        return self.nombre

class Producto(models.Model):
    nombre = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    descripcion = models.TextField()
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    categoria = models.ForeignKey(
        Categoria,
        on_delete=models.CASCADE,
        related_name='productos'
    )
    activo = models.BooleanField(default=True)
    creado_en = models.DateTimeField(auto_now_add=True)
    actualizado_en = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-creado_en']
        indexes = [
            models.Index(fields=['categoria', '-creado_en']),
            models.Index(fields=['slug']),
        ]
    
    def __str__(self):
        return self.nombre
    
    def tiene_stock(self):
        return self.stock > 0
    
    def reducir_stock(self, cantidad):
        if self.stock < cantidad:
            raise ValueError('Stock insuficiente')
        self.stock -= cantidad
        self.save()

class Pedido(models.Model):
    class Estado(models.TextChoices):
        PENDIENTE = 'pendiente', 'Pendiente'
        PAGADO = 'pagado', 'Pagado'
        ENVIADO = 'enviado', 'Enviado'
        ENTREGADO = 'entregado', 'Entregado'
        CANCELADO = 'cancelado', 'Cancelado'
    
    usuario = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='pedidos'
    )
    estado = models.CharField(
        max_length=20,
        choices=Estado.choices,
        default=Estado.PENDIENTE
    )
    total = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    creado_en = models.DateTimeField(auto_now_add=True)
    actualizado_en = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-creado_en']
    
    def __str__(self):
        return f'Pedido #{self.id} - {self.usuario.username}'
    
    def calcular_total(self):
        """Calcular total del pedido"""
        from django.db.models import Sum, F
        
        total = self.items.aggregate(
            total=Sum(F('precio') * F('cantidad'))
        )['total'] or 0
        
        self.total = total
        self.save()
        return total

class ItemPedido(models.Model):
    pedido = models.ForeignKey(
        Pedido,
        on_delete=models.CASCADE,
        related_name='items'
    )
    producto = models.ForeignKey(
        Producto,
        on_delete=models.PROTECT
    )
    cantidad = models.IntegerField(default=1)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    
    class Meta:
        unique_together = ['pedido', 'producto']
    
    def __str__(self):
        return f'{self.cantidad}x {self.producto.nombre}'
    
    def save(self, *args, **kwargs):
        # Guardar precio actual del producto
        if not self.precio:
            self.precio = self.producto.precio
        
        super().save(*args, **kwargs)
    
    @property
    def subtotal(self):
        return self.precio * self.cantidad
```

**Uso:**

```python
# Crear pedido
from django.contrib.auth.models import User
from django.db import transaction

usuario = User.objects.get(username='ana')
producto1 = Producto.objects.get(slug='laptop')
producto2 = Producto.objects.get(slug='mouse')

with transaction.atomic():
    # Crear pedido
    pedido = Pedido.objects.create(usuario=usuario)
    
    # Agregar items
    ItemPedido.objects.create(
        pedido=pedido,
        producto=producto1,
        cantidad=1
    )
    ItemPedido.objects.create(
        pedido=pedido,
        producto=producto2,
        cantidad=2
    )
    
    # Calcular total
    pedido.calcular_total()
    
    # Reducir stock
    for item in pedido.items.all():
        item.producto.reducir_stock(item.cantidad)

# Consultas optimizadas
pedidos = Pedido.objects.select_related('usuario').prefetch_related(
    'items__producto'
).filter(estado=Pedido.Estado.PAGADO)

for pedido in pedidos:
    print(f'Pedido de {pedido.usuario.username}:')
    for item in pedido.items.all():
        print(f'  {item.cantidad}x {item.producto.nombre} = ${item.subtotal}')
    print(f'Total: ${pedido.total}')
```

---

## Resumen

Has aprendido:

✅ Tipos de campos de Django  
✅ Relaciones: ForeignKey, OneToOne, ManyToMany  
✅ QuerySets: filter, exclude, get  
✅ Lookups avanzados  
✅ select_related y prefetch_related  
✅ Agregaciones y anotaciones  
✅ Transacciones

**Siguiente:** Views y URLs

---

## Recursos

- **[Model Field Reference](https://docs.djangoproject.com/en/5.0/ref/models/fields/)** - Todos los campos
- **[QuerySet API](https://docs.djangoproject.com/en/5.0/ref/models/querysets/)** - Métodos de QuerySet
- **[Database Optimization](https://docs.djangoproject.com/en/5.0/topics/db/optimization/)** - Optimización

El ORM de Django es poderoso. 🚀

# Django Admin Panel

> **Panel de administración automático y personalizable**

---

## ¿Qué es Django Admin?

**Panel completo de administración** generado automáticamente:
- ✅ CRUD para todos tus modelos
- ✅ Interfaz gráfica profesional
- ✅ Búsqueda, filtros, paginación
- ✅ Validaciones automáticas
- ✅ Sin escribir frontend

```python
# models.py
class Usuario(models.Model):
    nombre = models.CharField(max_length=100)
    email = models.EmailField()

# admin.py
admin.site.register(Usuario)

# ¡Ya tienes panel completo!
```

---

## Activar admin

**Ya viene incluido:**

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',  # ✅ Ya incluido
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

# urls.py
from django.contrib import admin

urlpatterns = [
    path('admin/', admin.site.urls),  # ✅ Ya incluido
]
```

**Crear superusuario:**

```bash
python manage.py createsuperuser

# Username: admin
# Email: admin@example.com
# Password: 
# Password (again):
# Superuser created successfully.
```

**Acceder:**
- URL: http://localhost:8000/admin
- Login con credenciales creadas

---

## Registro básico

```python
# admin.py
from django.contrib import admin
from .models import Usuario

# Registro simple
admin.site.register(Usuario)
```

---

## ModelAdmin personalizado

### Básico

```python
from django.contrib import admin
from .models import Usuario

@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    """Admin personalizado para Usuario"""
    
    # Columnas a mostrar
    list_display = ['id', 'nombre', 'email', 'activo', 'creado_en']
    
    # Filtros laterales
    list_filter = ['activo', 'creado_en']
    
    # Búsqueda
    search_fields = ['nombre', 'email']
    
    # Orden por defecto
    ordering = ['-creado_en']
    
    # Campos read-only
    readonly_fields = ['creado_en', 'actualizado_en']
```

### list_display avanzado

```python
@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = [
        'id',
        'nombre',
        'precio_formateado',
        'stock',
        'tiene_stock_display',
        'activo',
        'categoria'
    ]
    
    def precio_formateado(self, obj):
        """Precio con formato"""
        return f'${obj.precio:.2f}'
    precio_formateado.short_description = 'Precio'
    
    def tiene_stock_display(self, obj):
        """Indicador visual de stock"""
        if obj.stock > 10:
            return '✅ Sí'
        elif obj.stock > 0:
            return '⚠️ Poco'
        return '❌ No'
    tiene_stock_display.short_description = 'Stock'
```

### Colores en list_display

```python
from django.utils.html import format_html

@admin.register(Pedido)
class PedidoAdmin(admin.ModelAdmin):
    list_display = ['id', 'usuario', 'total', 'estado_coloreado', 'creado_en']
    
    def estado_coloreado(self, obj):
        """Estado con colores"""
        colores = {
            'pendiente': 'orange',
            'pagado': 'blue',
            'enviado': 'purple',
            'entregado': 'green',
            'cancelado': 'red'
        }
        
        color = colores.get(obj.estado, 'black')
        
        return format_html(
            '<span style="color: {}; font-weight: bold;">{}</span>',
            color,
            obj.get_estado_display()
        )
    estado_coloreado.short_description = 'Estado'
```

### list_editable

```python
@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ['id', 'nombre', 'precio', 'stock', 'activo']
    
    # Editar directamente desde la lista
    list_editable = ['precio', 'stock', 'activo']
```

### list_filter avanzado

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_filter = [
        'publicado',
        'creado_en',
        ('autor', admin.RelatedOnlyFieldListFilter),  # Solo autores con posts
    ]
```

### search_fields

```python
@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    # Búsqueda exacta
    search_fields = ['nombre', 'email']
    
    # Búsqueda con lookups
    search_fields = [
        'nombre__icontains',  # Insensible a mayúsculas
        '=email',  # Exacto
        '^username',  # Empieza con
    ]
    
    # Búsqueda en relaciones
    search_fields = ['nombre', 'pedidos__id']
```

---

## fieldsets (organizar formulario)

```python
@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    fieldsets = [
        ('Información básica', {
            'fields': ['nombre', 'email', 'telefono']
        }),
        ('Credenciales', {
            'fields': ['username', 'password'],
            'classes': ['collapse']  # Colapsado por defecto
        }),
        ('Estado', {
            'fields': ['activo', 'verificado'],
            'description': 'Configuración del estado del usuario'
        }),
        ('Fechas', {
            'fields': ['creado_en', 'actualizado_en'],
            'classes': ['collapse']
        }),
    ]
    
    readonly_fields = ['creado_en', 'actualizado_en']
```

---

## Inline Models

### TabularInline

```python
from django.contrib import admin
from .models import Pedido, ItemPedido

class ItemPedidoInline(admin.TabularInline):
    """Inline de items (tabla)"""
    model = ItemPedido
    extra = 1  # Formularios vacíos adicionales
    fields = ['producto', 'cantidad', 'precio']
    readonly_fields = ['precio']

@admin.register(Pedido)
class PedidoAdmin(admin.ModelAdmin):
    list_display = ['id', 'usuario', 'total', 'estado', 'creado_en']
    inlines = [ItemPedidoInline]
```

### StackedInline

```python
class PerfilInline(admin.StackedInline):
    """Inline apilado (vertical)"""
    model = Perfil
    fields = ['biografia', 'avatar', 'fecha_nacimiento']

@admin.register(User)
class UserAdmin(admin.ModelAdmin):
    inlines = [PerfilInline]
```

---

## Acciones personalizadas

```python
@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    list_display = ['id', 'nombre', 'email', 'activo']
    actions = ['activar_usuarios', 'desactivar_usuarios', 'enviar_email']
    
    @admin.action(description='Activar usuarios seleccionados')
    def activar_usuarios(self, request, queryset):
        """Activar múltiples usuarios"""
        count = queryset.update(activo=True)
        self.message_user(request, f'{count} usuarios activados')
    
    @admin.action(description='Desactivar usuarios seleccionados')
    def desactivar_usuarios(self, request, queryset):
        """Desactivar múltiples usuarios"""
        count = queryset.update(activo=False)
        self.message_user(request, f'{count} usuarios desactivados')
    
    @admin.action(description='Enviar email a seleccionados')
    def enviar_email(self, request, queryset):
        """Enviar email masivo"""
        from django.core.mail import send_mass_mail
        
        emails = [
            (
                'Asunto',
                'Mensaje',
                'noreply@tuapp.com',
                [usuario.email]
            )
            for usuario in queryset
        ]
        
        send_mass_mail(emails)
        self.message_user(request, f'Emails enviados a {len(emails)} usuarios')
```

---

## Override de métodos

### save_model

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    def save_model(self, request, obj, form, change):
        """Override save"""
        if not change:  # Solo al crear
            obj.autor = request.user
        
        super().save_model(request, obj, form, change)
```

### delete_model

```python
@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    def delete_model(self, request, obj):
        """Override delete"""
        # Soft delete
        obj.activo = False
        obj.save()
        
        # O realmente eliminar
        # super().delete_model(request, obj)
```

### get_queryset

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    def get_queryset(self, request):
        """Filtrar queryset"""
        qs = super().get_queryset(request)
        
        # Staff solo ve sus propios posts
        if not request.user.is_superuser:
            qs = qs.filter(autor=request.user)
        
        return qs
```

### has_add_permission / has_change_permission / has_delete_permission

```python
@admin.register(Configuracion)
class ConfiguracionAdmin(admin.ModelAdmin):
    def has_add_permission(self, request):
        """No permitir crear más de una configuración"""
        return not Configuracion.objects.exists()
    
    def has_delete_permission(self, request, obj=None):
        """No permitir eliminar"""
        return False
```

---

## Filtros personalizados

```python
from django.contrib import admin

class StockFilter(admin.SimpleListFilter):
    """Filtro personalizado de stock"""
    title = 'Stock'
    parameter_name = 'stock'
    
    def lookups(self, request, model_admin):
        """Opciones del filtro"""
        return [
            ('sin_stock', 'Sin stock'),
            ('poco_stock', 'Poco stock (< 10)'),
            ('con_stock', 'Con stock'),
        ]
    
    def queryset(self, request, queryset):
        """Filtrar queryset"""
        if self.value() == 'sin_stock':
            return queryset.filter(stock=0)
        
        if self.value() == 'poco_stock':
            return queryset.filter(stock__gt=0, stock__lt=10)
        
        if self.value() == 'con_stock':
            return queryset.filter(stock__gte=10)

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_filter = [StockFilter, 'activo', 'categoria']
```

---

## Personalizar URLs

```python
from django.urls import path

@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    def get_urls(self):
        """Agregar URLs personalizadas"""
        urls = super().get_urls()
        
        custom_urls = [
            path('importar/', self.admin_site.admin_view(self.importar_view), name='usuario_importar'),
            path('exportar/', self.admin_site.admin_view(self.exportar_view), name='usuario_exportar'),
        ]
        
        return custom_urls + urls
    
    def importar_view(self, request):
        """Vista personalizada de importar"""
        # ...
        pass
    
    def exportar_view(self, request):
        """Vista personalizada de exportar"""
        # ...
        pass
```

---

## Widgets personalizados

```python
from django import forms
from django.contrib import admin

class UsuarioAdminForm(forms.ModelForm):
    """Formulario personalizado"""
    biografia = forms.CharField(
        widget=forms.Textarea(attrs={'rows': 5, 'cols': 80})
    )
    
    class Meta:
        model = Usuario
        fields = '__all__'

@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    form = UsuarioAdminForm
```

---

## Permisos en Admin

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    def has_view_permission(self, request, obj=None):
        """Ver todos"""
        return True
    
    def has_change_permission(self, request, obj=None):
        """Editar solo propios"""
        if obj and obj.autor != request.user:
            return False
        return True
    
    def has_delete_permission(self, request, obj=None):
        """Eliminar solo si es admin o autor"""
        if obj:
            return request.user.is_superuser or obj.autor == request.user
        return True
```

---

## Customizar apariencia

### Título y header

```python
# admin.py
from django.contrib import admin

admin.site.site_header = 'Mi Aplicación Admin'
admin.site.site_title = 'Admin'
admin.site.index_title = 'Panel de Administración'
```

### CSS personalizado

```python
@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    class Media:
        css = {
            'all': ['admin/css/custom.css']
        }
        js = ['admin/js/custom.js']
```

---

## Ejemplo completo: E-commerce Admin

```python
from django.contrib import admin
from django.utils.html import format_html
from django.db.models import Sum, Count, F
from .models import Categoria, Producto, Pedido, ItemPedido

@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    """Admin de Categorías"""
    list_display = ['id', 'nombre', 'slug', 'total_productos']
    prepopulated_fields = {'slug': ('nombre',)}
    search_fields = ['nombre']
    
    def total_productos(self, obj):
        """Total de productos en la categoría"""
        return obj.productos.count()
    total_productos.short_description = 'Productos'

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    """Admin de Productos"""
    list_display = [
        'id',
        'nombre',
        'precio_formateado',
        'stock',
        'stock_status',
        'categoria',
        'activo',
        'imagen_thumbnail'
    ]
    list_filter = ['activo', 'categoria', 'creado_en']
    list_editable = ['precio', 'stock', 'activo']
    search_fields = ['nombre', 'descripcion']
    prepopulated_fields = {'slug': ('nombre',)}
    readonly_fields = ['creado_en', 'actualizado_en', 'imagen_preview']
    
    fieldsets = [
        ('Información básica', {
            'fields': ['nombre', 'slug', 'descripcion', 'categoria']
        }),
        ('Precio y Stock', {
            'fields': ['precio', 'stock', 'activo']
        }),
        ('Imagen', {
            'fields': ['imagen', 'imagen_preview']
        }),
        ('Fechas', {
            'fields': ['creado_en', 'actualizado_en'],
            'classes': ['collapse']
        }),
    ]
    
    actions = ['marcar_sin_stock', 'duplicar_productos']
    
    def precio_formateado(self, obj):
        """Precio con formato"""
        return f'${obj.precio:.2f}'
    precio_formateado.short_description = 'Precio'
    precio_formateado.admin_order_field = 'precio'
    
    def stock_status(self, obj):
        """Estado del stock con colores"""
        if obj.stock == 0:
            color = 'red'
            texto = '❌ Sin stock'
        elif obj.stock < 10:
            color = 'orange'
            texto = f'⚠️ Poco ({obj.stock})'
        else:
            color = 'green'
            texto = f'✅ Bien ({obj.stock})'
        
        return format_html(
            '<span style="color: {}; font-weight: bold;">{}</span>',
            color,
            texto
        )
    stock_status.short_description = 'Estado Stock'
    
    def imagen_thumbnail(self, obj):
        """Thumbnail de la imagen"""
        if obj.imagen:
            return format_html(
                '<img src="{}" width="50" height="50" style="object-fit: cover;" />',
                obj.imagen.url
            )
        return '-'
    imagen_thumbnail.short_description = 'Imagen'
    
    def imagen_preview(self, obj):
        """Preview grande de la imagen"""
        if obj.imagen:
            return format_html(
                '<img src="{}" width="300" />',
                obj.imagen.url
            )
        return 'Sin imagen'
    imagen_preview.short_description = 'Preview'
    
    @admin.action(description='Marcar sin stock')
    def marcar_sin_stock(self, request, queryset):
        """Poner stock en 0"""
        count = queryset.update(stock=0, activo=False)
        self.message_user(request, f'{count} productos marcados sin stock')
    
    @admin.action(description='Duplicar productos')
    def duplicar_productos(self, request, queryset):
        """Duplicar productos seleccionados"""
        for producto in queryset:
            producto.pk = None
            producto.nombre = f'{producto.nombre} (copia)'
            producto.slug = f'{producto.slug}-copia'
            producto.save()
        
        self.message_user(request, f'{queryset.count()} productos duplicados')

class ItemPedidoInline(admin.TabularInline):
    """Inline de items del pedido"""
    model = ItemPedido
    extra = 0
    readonly_fields = ['precio', 'subtotal_display']
    fields = ['producto', 'cantidad', 'precio', 'subtotal_display']
    
    def subtotal_display(self, obj):
        """Subtotal del item"""
        if obj.pk:
            return f'${obj.subtotal:.2f}'
        return '-'
    subtotal_display.short_description = 'Subtotal'

@admin.register(Pedido)
class PedidoAdmin(admin.ModelAdmin):
    """Admin de Pedidos"""
    list_display = [
        'id',
        'usuario',
        'total_formateado',
        'estado_coloreado',
        'total_items',
        'creado_en'
    ]
    list_filter = ['estado', 'creado_en']
    search_fields = ['id', 'usuario__username', 'usuario__email']
    readonly_fields = ['creado_en', 'actualizado_en', 'total']
    inlines = [ItemPedidoInline]
    
    fieldsets = [
        ('Información del pedido', {
            'fields': ['usuario', 'estado', 'total']
        }),
        ('Fechas', {
            'fields': ['creado_en', 'actualizado_en']
        }),
    ]
    
    actions = ['marcar_pagado', 'marcar_enviado', 'cancelar_pedidos']
    
    def total_formateado(self, obj):
        """Total con formato"""
        return f'${obj.total:.2f}'
    total_formateado.short_description = 'Total'
    total_formateado.admin_order_field = 'total'
    
    def estado_coloreado(self, obj):
        """Estado con colores"""
        colores = {
            'pendiente': '#ff9800',
            'pagado': '#2196f3',
            'enviado': '#9c27b0',
            'entregado': '#4caf50',
            'cancelado': '#f44336'
        }
        
        color = colores.get(obj.estado, '#000')
        
        return format_html(
            '<span style="background: {}; color: white; padding: 5px 10px; border-radius: 3px;">{}</span>',
            color,
            obj.get_estado_display()
        )
    estado_coloreado.short_description = 'Estado'
    
    def total_items(self, obj):
        """Total de items"""
        return obj.items.count()
    total_items.short_description = 'Items'
    
    @admin.action(description='Marcar como PAGADO')
    def marcar_pagado(self, request, queryset):
        """Marcar pedidos como pagados"""
        count = queryset.filter(estado='pendiente').update(estado='pagado')
        self.message_user(request, f'{count} pedidos marcados como pagados')
    
    @admin.action(description='Marcar como ENVIADO')
    def marcar_enviado(self, request, queryset):
        """Marcar pedidos como enviados"""
        count = queryset.filter(estado='pagado').update(estado='enviado')
        self.message_user(request, f'{count} pedidos marcados como enviados')
    
    @admin.action(description='Cancelar pedidos')
    def cancelar_pedidos(self, request, queryset):
        """Cancelar pedidos"""
        count = queryset.exclude(estado__in=['entregado', 'cancelado']).update(estado='cancelado')
        self.message_user(request, f'{count} pedidos cancelados')
    
    def get_queryset(self, request):
        """Optimizar queryset"""
        qs = super().get_queryset(request)
        return qs.select_related('usuario').annotate(
            num_items=Count('items')
        )

# Personalizar admin site
admin.site.site_header = 'E-commerce Admin'
admin.site.site_title = 'Admin'
admin.site.index_title = 'Panel de Administración'
```

---

## Resumen

Has aprendido:

✅ Activar y acceder al admin  
✅ ModelAdmin personalizado  
✅ list_display, list_filter, search_fields  
✅ fieldsets para organizar formularios  
✅ Inline models (TabularInline, StackedInline)  
✅ Acciones personalizadas  
✅ Filtros personalizados  
✅ Permisos en admin

**Siguiente:** Testing en Django

---

## Recursos

- **[Django Admin](https://docs.djangoproject.com/en/5.0/ref/contrib/admin/)** - Documentación oficial
- **[ModelAdmin options](https://docs.djangoproject.com/en/5.0/ref/contrib/admin/#modeladmin-options)** - Todas las opciones
- **[Admin actions](https://docs.djangoproject.com/en/5.0/ref/contrib/admin/actions/)** - Acciones personalizadas

El admin de Django es poderoso. 🎛️

# Django Views y URLs

> **El corazón de la lógica de tu aplicación web**

---

## ¿Qué son las Views?

**Views** = Funciones/clases que:
1. Reciben una request HTTP
2. Ejecutan lógica (consultar BD, procesar datos)
3. Retornan una response HTTP

```python
def mi_vista(request):
    # 1. Recibir request
    # 2. Lógica
    # 3. Retornar response
    return HttpResponse('Hola mundo')
```

---

## Function-Based Views (FBV)

### Vista básica

```python
from django.http import HttpResponse

def hola(request):
    """Vista más simple"""
    return HttpResponse('¡Hola mundo!')
```

### Con parámetros de URL

```python
def usuario_detail(request, id):
    """Vista con parámetro"""
    from .models import Usuario
    
    usuario = Usuario.objects.get(id=id)
    html = f'<h1>{usuario.nombre}</h1>'
    return HttpResponse(html)
```

### Con método HTTP

```python
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods

@require_http_methods(['GET'])
def listar_usuarios(request):
    """Solo GET"""
    from .models import Usuario
    
    usuarios = Usuario.objects.all()
    data = {
        'usuarios': [
            {'id': u.id, 'nombre': u.nombre}
            for u in usuarios
        ]
    }
    return JsonResponse(data)

@require_http_methods(['POST'])
def crear_usuario(request):
    """Solo POST"""
    import json
    from .models import Usuario
    
    datos = json.loads(request.body)
    usuario = Usuario.objects.create(
        nombre=datos['nombre'],
        email=datos['email']
    )
    
    return JsonResponse({
        'id': usuario.id,
        'nombre': usuario.nombre,
        'email': usuario.email
    }, status=201)
```

### Vista CRUD completa

```python
from django.http import JsonResponse, HttpResponseNotAllowed
from django.views.decorators.csrf import csrf_exempt
import json

from .models import Usuario

@csrf_exempt
def usuario_list_create(request):
    """GET /usuarios/ y POST /usuarios/"""
    
    if request.method == 'GET':
        usuarios = Usuario.objects.all()
        data = {
            'usuarios': [
                {
                    'id': u.id,
                    'nombre': u.nombre,
                    'email': u.email,
                    'activo': u.activo
                }
                for u in usuarios
            ],
            'total': usuarios.count()
        }
        return JsonResponse(data)
    
    elif request.method == 'POST':
        datos = json.loads(request.body)
        
        usuario = Usuario.objects.create(
            nombre=datos['nombre'],
            email=datos['email']
        )
        
        return JsonResponse({
            'id': usuario.id,
            'nombre': usuario.nombre,
            'email': usuario.email
        }, status=201)
    
    else:
        return HttpResponseNotAllowed(['GET', 'POST'])

@csrf_exempt
def usuario_detail(request, id):
    """GET/PUT/DELETE /usuarios/<id>/"""
    
    try:
        usuario = Usuario.objects.get(id=id)
    except Usuario.DoesNotExist:
        return JsonResponse({'error': 'Usuario no encontrado'}, status=404)
    
    if request.method == 'GET':
        return JsonResponse({
            'id': usuario.id,
            'nombre': usuario.nombre,
            'email': usuario.email,
            'activo': usuario.activo
        })
    
    elif request.method == 'PUT':
        datos = json.loads(request.body)
        
        usuario.nombre = datos.get('nombre', usuario.nombre)
        usuario.email = datos.get('email', usuario.email)
        usuario.activo = datos.get('activo', usuario.activo)
        usuario.save()
        
        return JsonResponse({
            'id': usuario.id,
            'nombre': usuario.nombre,
            'email': usuario.email
        })
    
    elif request.method == 'DELETE':
        usuario.delete()
        return JsonResponse({'mensaje': 'Usuario eliminado'}, status=204)
    
    else:
        return HttpResponseNotAllowed(['GET', 'PUT', 'DELETE'])
```

---

## Class-Based Views (CBV)

### Vista básica

```python
from django.views import View
from django.http import JsonResponse

class HolaView(View):
    """Vista basada en clase"""
    
    def get(self, request):
        return JsonResponse({'mensaje': 'Hola mundo'})
```

### CRUD con CBV

```python
from django.views import View
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator
import json

from .models import Usuario

@method_decorator(csrf_exempt, name='dispatch')
class UsuarioListCreateView(View):
    """GET y POST para usuarios"""
    
    def get(self, request):
        """Listar usuarios"""
        usuarios = Usuario.objects.all()
        
        data = {
            'usuarios': [
                {
                    'id': u.id,
                    'nombre': u.nombre,
                    'email': u.email
                }
                for u in usuarios
            ],
            'total': usuarios.count()
        }
        
        return JsonResponse(data)
    
    def post(self, request):
        """Crear usuario"""
        datos = json.loads(request.body)
        
        usuario = Usuario.objects.create(
            nombre=datos['nombre'],
            email=datos['email']
        )
        
        return JsonResponse({
            'id': usuario.id,
            'nombre': usuario.nombre,
            'email': usuario.email
        }, status=201)

@method_decorator(csrf_exempt, name='dispatch')
class UsuarioDetailView(View):
    """GET, PUT, DELETE para un usuario"""
    
    def get_object(self, id):
        """Helper para obtener usuario"""
        try:
            return Usuario.objects.get(id=id)
        except Usuario.DoesNotExist:
            return None
    
    def get(self, request, id):
        """Obtener usuario"""
        usuario = self.get_object(id)
        
        if not usuario:
            return JsonResponse({'error': 'No encontrado'}, status=404)
        
        return JsonResponse({
            'id': usuario.id,
            'nombre': usuario.nombre,
            'email': usuario.email
        })
    
    def put(self, request, id):
        """Actualizar usuario"""
        usuario = self.get_object(id)
        
        if not usuario:
            return JsonResponse({'error': 'No encontrado'}, status=404)
        
        datos = json.loads(request.body)
        usuario.nombre = datos.get('nombre', usuario.nombre)
        usuario.email = datos.get('email', usuario.email)
        usuario.save()
        
        return JsonResponse({
            'id': usuario.id,
            'nombre': usuario.nombre,
            'email': usuario.email
        })
    
    def delete(self, request, id):
        """Eliminar usuario"""
        usuario = self.get_object(id)
        
        if not usuario:
            return JsonResponse({'error': 'No encontrado'}, status=404)
        
        usuario.delete()
        return JsonResponse({'mensaje': 'Eliminado'}, status=204)
```

---

## Generic Views

### ListView

```python
from django.views.generic import ListView
from .models import Usuario

class UsuarioListView(ListView):
    """Lista de usuarios con paginación"""
    model = Usuario
    template_name = 'usuarios/lista.html'
    context_object_name = 'usuarios'
    paginate_by = 10
    
    def get_queryset(self):
        """Customizar queryset"""
        queryset = super().get_queryset()
        
        # Filtrar solo activos
        queryset = queryset.filter(activo=True)
        
        # Búsqueda
        q = self.request.GET.get('q')
        if q:
            queryset = queryset.filter(nombre__icontains=q)
        
        return queryset
    
    def get_context_data(self, **kwargs):
        """Agregar contexto adicional"""
        context = super().get_context_data(**kwargs)
        context['total'] = self.get_queryset().count()
        return context
```

### DetailView

```python
from django.views.generic import DetailView
from .models import Usuario

class UsuarioDetailView(DetailView):
    """Detalle de usuario"""
    model = Usuario
    template_name = 'usuarios/detalle.html'
    context_object_name = 'usuario'
```

### CreateView

```python
from django.views.generic.edit import CreateView
from django.urls import reverse_lazy
from .models import Usuario
from .forms import UsuarioForm

class UsuarioCreateView(CreateView):
    """Crear usuario"""
    model = Usuario
    form_class = UsuarioForm
    template_name = 'usuarios/crear.html'
    success_url = reverse_lazy('usuario_list')
    
    def form_valid(self, form):
        """Ejecutar antes de guardar"""
        usuario = form.save(commit=False)
        # Lógica adicional
        usuario.save()
        return super().form_valid(form)
```

### UpdateView

```python
from django.views.generic.edit import UpdateView
from django.urls import reverse_lazy
from .models import Usuario
from .forms import UsuarioForm

class UsuarioUpdateView(UpdateView):
    """Actualizar usuario"""
    model = Usuario
    form_class = UsuarioForm
    template_name = 'usuarios/editar.html'
    success_url = reverse_lazy('usuario_list')
```

### DeleteView

```python
from django.views.generic.edit import DeleteView
from django.urls import reverse_lazy
from .models import Usuario

class UsuarioDeleteView(DeleteView):
    """Eliminar usuario"""
    model = Usuario
    template_name = 'usuarios/confirmar_eliminar.html'
    success_url = reverse_lazy('usuario_list')
```

---

## URLs

### urls.py básico

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.hola, name='hola'),
]
```

### URLs con parámetros

```python
from django.urls import path
from . import views

urlpatterns = [
    # /usuarios/
    path('usuarios/', views.usuario_list, name='usuario_list'),
    
    # /usuarios/5/
    path('usuarios/<int:id>/', views.usuario_detail, name='usuario_detail'),
    
    # /usuarios/ana/
    path('usuarios/<str:username>/', views.usuario_por_username),
    
    # /posts/2024/12/
    path('posts/<int:year>/<int:month>/', views.posts_por_fecha),
]
```

### Convertidores de path

```python
# int: números enteros
path('usuarios/<int:id>/', views.usuario_detail)

# str: cualquier string (sin /)
path('usuarios/<str:username>/', views.usuario_por_username)

# slug: letras, números, guiones, guiones bajos
path('posts/<slug:slug>/', views.post_detail)

# uuid: UUID
path('pedidos/<uuid:id>/', views.pedido_detail)

# path: cualquier string (incluye /)
path('archivos/<path:ruta>/', views.archivo)
```

### Convertidores personalizados

```python
# converters.py
class YearConverter:
    regex = '[0-9]{4}'
    
    def to_python(self, value):
        return int(value)
    
    def to_url(self, value):
        return str(value)

# urls.py
from django.urls import path, register_converter
from . import converters, views

register_converter(converters.YearConverter, 'year')

urlpatterns = [
    path('posts/<year:year>/', views.posts_por_year),
]
```

### Incluir URLs de apps

**proyecto/urls.py** (principal)

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/usuarios/', include('usuarios.urls')),
    path('api/productos/', include('productos.urls')),
]
```

**usuarios/urls.py** (app)

```python
from django.urls import path
from . import views

app_name = 'usuarios'  # Namespace

urlpatterns = [
    path('', views.usuario_list, name='list'),
    path('<int:id>/', views.usuario_detail, name='detail'),
    path('crear/', views.usuario_create, name='create'),
]
```

---

## reverse() y get_absolute_url()

### reverse()

```python
from django.urls import reverse

# Sin parámetros
url = reverse('usuario_list')
# /api/usuarios/

# Con parámetros
url = reverse('usuario_detail', kwargs={'id': 5})
# /api/usuarios/5/

# Con namespace
url = reverse('usuarios:detail', kwargs={'id': 5})
# /api/usuarios/5/
```

### get_absolute_url()

```python
from django.db import models
from django.urls import reverse

class Usuario(models.Model):
    nombre = models.CharField(max_length=100)
    
    def get_absolute_url(self):
        """URL canónica del objeto"""
        return reverse('usuarios:detail', kwargs={'id': self.id})

# Uso
usuario = Usuario.objects.get(id=5)
print(usuario.get_absolute_url())  # /api/usuarios/5/
```

---

## Request object

```python
def mi_vista(request):
    # Método HTTP
    method = request.method  # 'GET', 'POST', 'PUT', etc.
    
    # Parámetros GET
    q = request.GET.get('q')
    page = request.GET.get('page', 1)
    
    # Datos POST
    nombre = request.POST.get('nombre')
    
    # Body (JSON)
    import json
    datos = json.loads(request.body)
    
    # Headers
    auth = request.headers.get('Authorization')
    
    # Usuario autenticado
    if request.user.is_authenticated:
        usuario = request.user
    
    # Cookies
    session_id = request.COOKIES.get('sessionid')
    
    # Archivos subidos
    archivo = request.FILES.get('archivo')
    
    # IP del cliente
    ip = request.META.get('REMOTE_ADDR')
    
    # URL completa
    url = request.build_absolute_uri()
    
    # Es AJAX?
    is_ajax = request.headers.get('X-Requested-With') == 'XMLHttpRequest'
    
    return HttpResponse('OK')
```

---

## Response types

### HttpResponse

```python
from django.http import HttpResponse

def vista(request):
    return HttpResponse('Texto plano')

def vista_html(request):
    html = '<html><body><h1>Hola</h1></body></html>'
    return HttpResponse(html)

def vista_con_status(request):
    return HttpResponse('Creado', status=201)
```

### JsonResponse

```python
from django.http import JsonResponse

def vista(request):
    data = {
        'nombre': 'Ana',
        'edad': 25
    }
    return JsonResponse(data)

def vista_lista(request):
    data = [1, 2, 3, 4, 5]
    return JsonResponse(data, safe=False)  # safe=False para listas
```

### HttpResponseRedirect

```python
from django.http import HttpResponseRedirect
from django.shortcuts import redirect

def vista(request):
    # Método 1
    return HttpResponseRedirect('/usuarios/')
    
    # Método 2 (mejor)
    return redirect('usuario_list')
    
    # Con parámetros
    return redirect('usuario_detail', id=5)
```

### FileResponse

```python
from django.http import FileResponse

def descargar_archivo(request, id):
    archivo = Archivo.objects.get(id=id)
    
    response = FileResponse(
        archivo.file.open('rb'),
        content_type='application/pdf'
    )
    response['Content-Disposition'] = f'attachment; filename="{archivo.nombre}"'
    
    return response
```

### StreamingHttpResponse

```python
from django.http import StreamingHttpResponse

def generar_csv():
    yield 'id,nombre,email\n'
    for usuario in Usuario.objects.all():
        yield f'{usuario.id},{usuario.nombre},{usuario.email}\n'

def exportar_usuarios(request):
    response = StreamingHttpResponse(
        generar_csv(),
        content_type='text/csv'
    )
    response['Content-Disposition'] = 'attachment; filename="usuarios.csv"'
    return response
```

---

## Shortcuts

### render()

```python
from django.shortcuts import render

def usuario_list(request):
    usuarios = Usuario.objects.all()
    
    return render(request, 'usuarios/lista.html', {
        'usuarios': usuarios,
        'total': usuarios.count()
    })
```

### get_object_or_404()

```python
from django.shortcuts import get_object_or_404

def usuario_detail(request, id):
    # En vez de try/except
    usuario = get_object_or_404(Usuario, id=id)
    
    return render(request, 'usuarios/detalle.html', {
        'usuario': usuario
    })
```

### get_list_or_404()

```python
from django.shortcuts import get_list_or_404

def usuarios_activos(request):
    usuarios = get_list_or_404(Usuario, activo=True)
    
    return render(request, 'usuarios/lista.html', {
        'usuarios': usuarios
    })
```

---

## Decoradores

### require_http_methods

```python
from django.views.decorators.http import (
    require_http_methods,
    require_GET,
    require_POST
)

@require_GET
def vista_get(request):
    return HttpResponse('Solo GET')

@require_POST
def vista_post(request):
    return HttpResponse('Solo POST')

@require_http_methods(['GET', 'POST'])
def vista_get_post(request):
    if request.method == 'GET':
        return HttpResponse('GET')
    else:
        return HttpResponse('POST')
```

### login_required

```python
from django.contrib.auth.decorators import login_required

@login_required
def perfil(request):
    """Solo usuarios autenticados"""
    return render(request, 'perfil.html', {
        'usuario': request.user
    })

@login_required(login_url='/login/')
def configuracion(request):
    """Redirigir a /login/ si no autenticado"""
    return render(request, 'configuracion.html')
```

### permission_required

```python
from django.contrib.auth.decorators import permission_required

@permission_required('usuarios.add_usuario')
def crear_usuario(request):
    """Requiere permiso add_usuario"""
    # ...
    pass

@permission_required(['usuarios.view_usuario', 'usuarios.change_usuario'])
def editar_usuario(request, id):
    """Requiere múltiples permisos"""
    # ...
    pass
```

### csrf_exempt

```python
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def api_webhook(request):
    """Desactivar CSRF para webhooks externos"""
    datos = json.loads(request.body)
    # Procesar webhook
    return JsonResponse({'status': 'ok'})
```

---

## Mixins (para CBV)

### LoginRequiredMixin

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView

class UsuarioListView(LoginRequiredMixin, ListView):
    """Solo usuarios autenticados"""
    model = Usuario
    login_url = '/login/'
```

### PermissionRequiredMixin

```python
from django.contrib.auth.mixins import PermissionRequiredMixin
from django.views.generic import CreateView

class UsuarioCreateView(PermissionRequiredMixin, CreateView):
    """Requiere permiso"""
    model = Usuario
    permission_required = 'usuarios.add_usuario'
```

### UserPassesTestMixin

```python
from django.contrib.auth.mixins import UserPassesTestMixin
from django.views.generic import UpdateView

class UsuarioUpdateView(UserPassesTestMixin, UpdateView):
    """Solo el propio usuario o admin"""
    model = Usuario
    
    def test_func(self):
        usuario = self.get_object()
        return self.request.user == usuario or self.request.user.is_staff
```

---

## Ejemplo completo: Blog API

**models.py**

```python
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    titulo = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    contenido = models.TextField()
    autor = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    publicado = models.BooleanField(default=False)
    creado_en = models.DateTimeField(auto_now_add=True)
    actualizado_en = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-creado_en']
    
    def __str__(self):
        return self.titulo
```

**views.py**

```python
from django.views import View
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator
from django.contrib.auth.decorators import login_required
from django.shortcuts import get_object_or_404
import json

from .models import Post

class PostListView(View):
    """GET /api/posts/"""
    
    def get(self, request):
        posts = Post.objects.filter(publicado=True)
        
        # Filtros opcionales
        autor_id = request.GET.get('autor')
        if autor_id:
            posts = posts.filter(autor_id=autor_id)
        
        # Búsqueda
        q = request.GET.get('q')
        if q:
            posts = posts.filter(titulo__icontains=q)
        
        # Paginación
        page = int(request.GET.get('page', 1))
        per_page = int(request.GET.get('per_page', 10))
        
        start = (page - 1) * per_page
        end = start + per_page
        
        total = posts.count()
        posts_pagina = posts[start:end]
        
        data = {
            'posts': [
                {
                    'id': p.id,
                    'titulo': p.titulo,
                    'slug': p.slug,
                    'contenido': p.contenido[:200],  # Resumen
                    'autor': {
                        'id': p.autor.id,
                        'username': p.autor.username
                    },
                    'creado_en': p.creado_en.isoformat()
                }
                for p in posts_pagina
            ],
            'total': total,
            'page': page,
            'per_page': per_page,
            'pages': (total + per_page - 1) // per_page
        }
        
        return JsonResponse(data)

@method_decorator(csrf_exempt, name='dispatch')
@method_decorator(login_required, name='dispatch')
class PostCreateView(View):
    """POST /api/posts/"""
    
    def post(self, request):
        datos = json.loads(request.body)
        
        post = Post.objects.create(
            titulo=datos['titulo'],
            slug=datos['slug'],
            contenido=datos['contenido'],
            autor=request.user,
            publicado=datos.get('publicado', False)
        )
        
        return JsonResponse({
            'id': post.id,
            'titulo': post.titulo,
            'slug': post.slug
        }, status=201)

class PostDetailView(View):
    """GET /api/posts/<slug>/"""
    
    def get(self, request, slug):
        post = get_object_or_404(Post, slug=slug, publicado=True)
        
        return JsonResponse({
            'id': post.id,
            'titulo': post.titulo,
            'slug': post.slug,
            'contenido': post.contenido,
            'autor': {
                'id': post.autor.id,
                'username': post.autor.username
            },
            'creado_en': post.creado_en.isoformat(),
            'actualizado_en': post.actualizado_en.isoformat()
        })

@method_decorator(csrf_exempt, name='dispatch')
@method_decorator(login_required, name='dispatch')
class PostUpdateView(View):
    """PUT /api/posts/<slug>/"""
    
    def put(self, request, slug):
        post = get_object_or_404(Post, slug=slug)
        
        # Solo el autor puede editar
        if post.autor != request.user:
            return JsonResponse({'error': 'No autorizado'}, status=403)
        
        datos = json.loads(request.body)
        
        post.titulo = datos.get('titulo', post.titulo)
        post.contenido = datos.get('contenido', post.contenido)
        post.publicado = datos.get('publicado', post.publicado)
        post.save()
        
        return JsonResponse({
            'id': post.id,
            'titulo': post.titulo,
            'slug': post.slug
        })

@method_decorator(csrf_exempt, name='dispatch')
@method_decorator(login_required, name='dispatch')
class PostDeleteView(View):
    """DELETE /api/posts/<slug>/"""
    
    def delete(self, request, slug):
        post = get_object_or_404(Post, slug=slug)
        
        # Solo el autor o admin puede eliminar
        if post.autor != request.user and not request.user.is_staff:
            return JsonResponse({'error': 'No autorizado'}, status=403)
        
        post.delete()
        return JsonResponse({'mensaje': 'Post eliminado'}, status=204)
```

**urls.py**

```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('posts/', views.PostListView.as_view(), name='post_list'),
    path('posts/crear/', views.PostCreateView.as_view(), name='post_create'),
    path('posts/<slug:slug>/', views.PostDetailView.as_view(), name='post_detail'),
    path('posts/<slug:slug>/editar/', views.PostUpdateView.as_view(), name='post_update'),
    path('posts/<slug:slug>/eliminar/', views.PostDeleteView.as_view(), name='post_delete'),
]
```

---

## Resumen

Has aprendido:

✅ Function-Based Views (FBV)  
✅ Class-Based Views (CBV)  
✅ Generic Views (ListView, DetailView, etc.)  
✅ URLs con parámetros  
✅ Request y Response objects  
✅ Decoradores (@login_required, etc.)  
✅ Mixins para CBV

**Siguiente:** Django REST Framework

---

## Recursos

- **[Views](https://docs.djangoproject.com/en/5.0/topics/http/views/)** - Documentación de vistas
- **[URL dispatcher](https://docs.djangoproject.com/en/5.0/topics/http/urls/)** - Routing
- **[Generic Views](https://docs.djangoproject.com/en/5.0/ref/class-based-views/)** - Vistas genéricas

Views controlan la lógica de tu app. 🎯

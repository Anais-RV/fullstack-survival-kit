# Django REST Framework (DRF)

> **Framework profesional para construir APIs REST con Django**

---

## ¿Por qué DRF?

**Django vanilla:**
```python
def usuario_list(request):
    usuarios = Usuario.objects.all()
    data = [{'id': u.id, 'nombre': u.nombre} for u in usuarios]
    return JsonResponse(data, safe=False)
```

**Con DRF:**
```python
class UsuarioViewSet(viewsets.ModelViewSet):
    queryset = Usuario.objects.all()
    serializer_class = UsuarioSerializer
    # ¡6 endpoints CRUD completos!
```

---

## Instalación

```bash
pip install djangorestframework

# Opcional pero recomendado
pip install markdown  # Para la API navegable
pip install django-filter  # Para filtros avanzados
```

**settings.py**

```python
INSTALLED_APPS = [
    # ...
    'rest_framework',
]

REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
}
```

---

## Serializers

### Serializer básico

```python
from rest_framework import serializers

class UsuarioSerializer(serializers.Serializer):
    """Serializer manual"""
    id = serializers.IntegerField(read_only=True)
    nombre = serializers.CharField(max_length=100)
    email = serializers.EmailField()
    edad = serializers.IntegerField(required=False)
    activo = serializers.BooleanField(default=True)
    
    def create(self, validated_data):
        """Crear usuario"""
        return Usuario.objects.create(**validated_data)
    
    def update(self, instance, validated_data):
        """Actualizar usuario"""
        instance.nombre = validated_data.get('nombre', instance.nombre)
        instance.email = validated_data.get('email', instance.email)
        instance.edad = validated_data.get('edad', instance.edad)
        instance.activo = validated_data.get('activo', instance.activo)
        instance.save()
        return instance
```

### ModelSerializer (automático)

```python
from rest_framework import serializers
from .models import Usuario

class UsuarioSerializer(serializers.ModelSerializer):
    """Serializer automático desde modelo"""
    
    class Meta:
        model = Usuario
        fields = ['id', 'nombre', 'email', 'edad', 'activo', 'creado_en']
        read_only_fields = ['id', 'creado_en']
```

**Todos los campos:**

```python
class UsuarioSerializer(serializers.ModelSerializer):
    class Meta:
        model = Usuario
        fields = '__all__'
```

**Excluir campos:**

```python
class UsuarioSerializer(serializers.ModelSerializer):
    class Meta:
        model = Usuario
        exclude = ['password', 'token']
```

---

## Validación en Serializers

### Validación de campo

```python
class UsuarioSerializer(serializers.ModelSerializer):
    class Meta:
        model = Usuario
        fields = ['id', 'nombre', 'email', 'edad']
    
    def validate_edad(self, value):
        """Validar edad específica"""
        if value < 18:
            raise serializers.ValidationError('Debe ser mayor de 18 años')
        return value
    
    def validate_email(self, value):
        """Validar email único"""
        if Usuario.objects.filter(email=value).exists():
            raise serializers.ValidationError('Email ya existe')
        return value
```

### Validación múltiple

```python
class UsuarioSerializer(serializers.ModelSerializer):
    class Meta:
        model = Usuario
        fields = ['id', 'nombre', 'email', 'password', 'password_confirm']
        extra_kwargs = {
            'password': {'write_only': True}
        }
    
    def validate(self, data):
        """Validar múltiples campos"""
        if data.get('password') != data.get('password_confirm'):
            raise serializers.ValidationError('Las contraseñas no coinciden')
        
        return data
    
    def create(self, validated_data):
        """Crear usuario con password hasheada"""
        validated_data.pop('password_confirm')
        password = validated_data.pop('password')
        
        usuario = Usuario.objects.create(**validated_data)
        usuario.set_password(password)
        usuario.save()
        
        return usuario
```

---

## Campos personalizados

### SerializerMethodField

```python
class UsuarioSerializer(serializers.ModelSerializer):
    # Campo calculado
    nombre_completo = serializers.SerializerMethodField()
    
    class Meta:
        model = Usuario
        fields = ['id', 'nombre', 'apellido', 'nombre_completo']
    
    def get_nombre_completo(self, obj):
        """Calcular nombre completo"""
        return f'{obj.nombre} {obj.apellido}'
```

### PrimaryKeyRelatedField

```python
class LibroSerializer(serializers.ModelSerializer):
    # Solo ID del autor
    autor = serializers.PrimaryKeyRelatedField(
        queryset=Autor.objects.all()
    )
    
    class Meta:
        model = Libro
        fields = ['id', 'titulo', 'autor']
```

### StringRelatedField

```python
class LibroSerializer(serializers.ModelSerializer):
    # Usa __str__ del modelo
    autor = serializers.StringRelatedField()
    
    class Meta:
        model = Libro
        fields = ['id', 'titulo', 'autor']
    
    # Output: {"id": 1, "titulo": "...", "autor": "García Márquez"}
```

### Nested Serializers

```python
class AutorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Autor
        fields = ['id', 'nombre', 'biografia']

class LibroSerializer(serializers.ModelSerializer):
    # Serializer anidado
    autor = AutorSerializer(read_only=True)
    
    class Meta:
        model = Libro
        fields = ['id', 'titulo', 'autor']
    
    # Output:
    # {
    #     "id": 1,
    #     "titulo": "...",
    #     "autor": {
    #         "id": 1,
    #         "nombre": "García Márquez",
    #         "biografia": "..."
    #     }
    # }
```

### Reverse relationships

```python
class AutorSerializer(serializers.ModelSerializer):
    # Incluir libros del autor
    libros = LibroSerializer(many=True, read_only=True)
    
    class Meta:
        model = Autor
        fields = ['id', 'nombre', 'libros']
```

---

## APIView

### APIView básica

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class UsuarioListView(APIView):
    """GET y POST /api/usuarios/"""
    
    def get(self, request):
        """Listar usuarios"""
        usuarios = Usuario.objects.all()
        serializer = UsuarioSerializer(usuarios, many=True)
        return Response(serializer.data)
    
    def post(self, request):
        """Crear usuario"""
        serializer = UsuarioSerializer(data=request.data)
        
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class UsuarioDetailView(APIView):
    """GET, PUT, DELETE /api/usuarios/<id>/"""
    
    def get_object(self, pk):
        """Helper para obtener usuario"""
        try:
            return Usuario.objects.get(pk=pk)
        except Usuario.DoesNotExist:
            return None
    
    def get(self, request, pk):
        """Obtener usuario"""
        usuario = self.get_object(pk)
        if not usuario:
            return Response({'error': 'No encontrado'}, status=status.HTTP_404_NOT_FOUND)
        
        serializer = UsuarioSerializer(usuario)
        return Response(serializer.data)
    
    def put(self, request, pk):
        """Actualizar usuario"""
        usuario = self.get_object(pk)
        if not usuario:
            return Response({'error': 'No encontrado'}, status=status.HTTP_404_NOT_FOUND)
        
        serializer = UsuarioSerializer(usuario, data=request.data)
        
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def delete(self, request, pk):
        """Eliminar usuario"""
        usuario = self.get_object(pk)
        if not usuario:
            return Response({'error': 'No encontrado'}, status=status.HTTP_404_NOT_FOUND)
        
        usuario.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

---

## Generic Views

### ListCreateAPIView

```python
from rest_framework import generics

class UsuarioListCreateView(generics.ListCreateAPIView):
    """GET y POST en una sola clase"""
    queryset = Usuario.objects.all()
    serializer_class = UsuarioSerializer
```

### RetrieveUpdateDestroyAPIView

```python
class UsuarioDetailView(generics.RetrieveUpdateDestroyAPIView):
    """GET, PUT, PATCH, DELETE en una sola clase"""
    queryset = Usuario.objects.all()
    serializer_class = UsuarioSerializer
```

### Todas las Generic Views

```python
from rest_framework import generics

# Solo lectura
generics.ListAPIView  # GET lista
generics.RetrieveAPIView  # GET detalle

# Solo escritura
generics.CreateAPIView  # POST
generics.UpdateAPIView  # PUT, PATCH
generics.DestroyAPIView  # DELETE

# Combinadas
generics.ListCreateAPIView  # GET lista + POST
generics.RetrieveUpdateAPIView  # GET + PUT + PATCH
generics.RetrieveDestroyAPIView  # GET + DELETE
generics.RetrieveUpdateDestroyAPIView  # GET + PUT + PATCH + DELETE
```

---

## ViewSets

### ModelViewSet (CRUD completo)

```python
from rest_framework import viewsets

class UsuarioViewSet(viewsets.ModelViewSet):
    """
    Genera automáticamente:
    - GET /usuarios/ (list)
    - POST /usuarios/ (create)
    - GET /usuarios/<pk>/ (retrieve)
    - PUT /usuarios/<pk>/ (update)
    - PATCH /usuarios/<pk>/ (partial_update)
    - DELETE /usuarios/<pk>/ (destroy)
    """
    queryset = Usuario.objects.all()
    serializer_class = UsuarioSerializer
```

### ReadOnlyModelViewSet

```python
class UsuarioViewSet(viewsets.ReadOnlyModelViewSet):
    """Solo lectura: list y retrieve"""
    queryset = Usuario.objects.all()
    serializer_class = UsuarioSerializer
```

### Acciones personalizadas

```python
from rest_framework.decorators import action
from rest_framework.response import Response

class UsuarioViewSet(viewsets.ModelViewSet):
    queryset = Usuario.objects.all()
    serializer_class = UsuarioSerializer
    
    @action(detail=False, methods=['get'])
    def activos(self, request):
        """GET /usuarios/activos/"""
        usuarios = self.queryset.filter(activo=True)
        serializer = self.get_serializer(usuarios, many=True)
        return Response(serializer.data)
    
    @action(detail=True, methods=['post'])
    def activar(self, request, pk=None):
        """POST /usuarios/<pk>/activar/"""
        usuario = self.get_object()
        usuario.activo = True
        usuario.save()
        return Response({'status': 'activado'})
    
    @action(detail=True, methods=['get'])
    def pedidos(self, request, pk=None):
        """GET /usuarios/<pk>/pedidos/"""
        usuario = self.get_object()
        pedidos = usuario.pedidos.all()
        # Usar otro serializer
        serializer = PedidoSerializer(pedidos, many=True)
        return Response(serializer.data)
```

---

## Routers

### SimpleRouter

```python
from rest_framework.routers import SimpleRouter
from . import views

router = SimpleRouter()
router.register('usuarios', views.UsuarioViewSet, basename='usuario')

urlpatterns = router.urls

# Genera:
# GET    /usuarios/           -> list
# POST   /usuarios/           -> create
# GET    /usuarios/<pk>/      -> retrieve
# PUT    /usuarios/<pk>/      -> update
# PATCH  /usuarios/<pk>/      -> partial_update
# DELETE /usuarios/<pk>/      -> destroy
```

### DefaultRouter

```python
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register('usuarios', views.UsuarioViewSet)
router.register('productos', views.ProductoViewSet)

urlpatterns = router.urls

# Genera root API con lista de endpoints:
# GET / -> {"usuarios": "...", "productos": "..."}
```

### Incluir en urls.py principal

```python
# proyecto/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from usuarios import views as usuario_views
from productos import views as producto_views

router = DefaultRouter()
router.register('usuarios', usuario_views.UsuarioViewSet)
router.register('productos', producto_views.ProductoViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
```

---

## Autenticación

### SessionAuthentication

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
    ],
}

# Usa las sesiones de Django
# El usuario debe hacer login via /admin o vista de login
```

### TokenAuthentication

**settings.py**

```python
INSTALLED_APPS = [
    # ...
    'rest_framework',
    'rest_framework.authtoken',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
}
```

**Migrar:**

```bash
python manage.py migrate
```

**Generar tokens:**

```python
from rest_framework.authtoken.models import Token
from django.contrib.auth.models import User

# Crear token para usuario
usuario = User.objects.get(username='ana')
token = Token.objects.create(user=usuario)
print(token.key)  # 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b

# O get_or_create
token, created = Token.objects.get_or_create(user=usuario)
```

**Vista de login:**

```python
from rest_framework.authtoken.models import Token
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import AllowAny
from rest_framework.response import Response
from django.contrib.auth import authenticate

@api_view(['POST'])
@permission_classes([AllowAny])
def login(request):
    username = request.data.get('username')
    password = request.data.get('password')
    
    user = authenticate(username=username, password=password)
    
    if user:
        token, _ = Token.objects.get_or_create(user=user)
        return Response({
            'token': token.key,
            'user_id': user.id,
            'username': user.username
        })
    
    return Response({'error': 'Credenciales inválidas'}, status=400)
```

**Uso del token:**

```bash
# Sin token
curl http://localhost:8000/api/usuarios/
# 401 Unauthorized

# Con token
curl http://localhost:8000/api/usuarios/ \
  -H "Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b"
# 200 OK
```

### JWT (Simple JWT)

**Instalación:**

```bash
pip install djangorestframework-simplejwt
```

**settings.py**

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}

from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
}
```

**urls.py**

```python
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

**Uso:**

```bash
# Obtener token
curl -X POST http://localhost:8000/api/token/ \
  -H "Content-Type: application/json" \
  -d '{"username": "ana", "password": "pass123"}'

# Respuesta:
# {
#   "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
#   "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
# }

# Usar access token
curl http://localhost:8000/api/usuarios/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..."

# Renovar access token
curl -X POST http://localhost:8000/api/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{"refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."}'
```

---

## Permisos

### Permisos globales

```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

### Permisos por vista

```python
from rest_framework.permissions import IsAuthenticated, AllowAny

class UsuarioViewSet(viewsets.ModelViewSet):
    queryset = Usuario.objects.all()
    serializer_class = UsuarioSerializer
    permission_classes = [IsAuthenticated]

class PublicView(APIView):
    permission_classes = [AllowAny]
    
    def get(self, request):
        return Response({'mensaje': 'Público'})
```

### Permisos incluidos

```python
from rest_framework.permissions import (
    AllowAny,  # Todos
    IsAuthenticated,  # Solo autenticados
    IsAdminUser,  # Solo admin
    IsAuthenticatedOrReadOnly,  # Auth para escribir, todos para leer
)
```

### Permisos personalizados

```python
from rest_framework.permissions import BasePermission

class IsOwnerOrReadOnly(BasePermission):
    """Solo el propietario puede editar"""
    
    def has_object_permission(self, request, view, obj):
        # Lectura permitida para todos
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        
        # Escritura solo para el propietario
        return obj.usuario == request.user

class IsAuthorOrAdmin(BasePermission):
    """Solo el autor o admin pueden editar/eliminar"""
    
    def has_object_permission(self, request, view, obj):
        # Lectura permitida
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        
        # Escritura para autor o admin
        return obj.autor == request.user or request.user.is_staff

# Uso
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticatedOrReadOnly, IsAuthorOrAdmin]
```

---

## Filtros

### django-filter

**Instalación:**

```bash
pip install django-filter
```

**settings.py**

```python
INSTALLED_APPS = [
    # ...
    'django_filters',
]

REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
}
```

**Filtros básicos:**

```python
from django_filters.rest_framework import DjangoFilterBackend

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['categoria', 'activo']

# GET /productos/?categoria=1
# GET /productos/?activo=true
```

**Filtros avanzados:**

```python
import django_filters

class ProductoFilter(django_filters.FilterSet):
    precio_min = django_filters.NumberFilter(field_name='precio', lookup_expr='gte')
    precio_max = django_filters.NumberFilter(field_name='precio', lookup_expr='lte')
    nombre = django_filters.CharFilter(lookup_expr='icontains')
    
    class Meta:
        model = Producto
        fields = ['categoria', 'activo']

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_class = ProductoFilter

# GET /productos/?precio_min=10&precio_max=100
# GET /productos/?nombre=laptop
```

### SearchFilter

```python
from rest_framework.filters import SearchFilter

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    filter_backends = [SearchFilter]
    search_fields = ['nombre', 'descripcion']

# GET /productos/?search=laptop
```

### OrderingFilter

```python
from rest_framework.filters import OrderingFilter

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    filter_backends = [OrderingFilter]
    ordering_fields = ['precio', 'nombre', 'creado_en']
    ordering = ['-creado_en']  # Default

# GET /productos/?ordering=precio
# GET /productos/?ordering=-precio
# GET /productos/?ordering=nombre,-precio
```

---

## Paginación

### PageNumberPagination

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}

# GET /productos/?page=1
# GET /productos/?page=2
```

**Respuesta:**

```json
{
  "count": 100,
  "next": "http://localhost:8000/productos/?page=2",
  "previous": null,
  "results": [...]
}
```

### LimitOffsetPagination

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 10,
}

# GET /productos/?limit=20&offset=40
```

### Paginación personalizada

```python
from rest_framework.pagination import PageNumberPagination

class CustomPagination(PageNumberPagination):
    page_size = 20
    page_size_query_param = 'per_page'
    max_page_size = 100

class ProductoViewSet(viewsets.ModelViewSet):
    queryset = Producto.objects.all()
    serializer_class = ProductoSerializer
    pagination_class = CustomPagination

# GET /productos/?page=2&per_page=50
```

---

## Throttling (Rate Limiting)

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day',
    }
}

# Por vista
class UsuarioViewSet(viewsets.ModelViewSet):
    throttle_classes = [UserRateThrottle]
    throttle_scope = 'uploads'
```

---

## Resumen

Has aprendido:

✅ Serializers (ModelSerializer)  
✅ APIView y Generic Views  
✅ ViewSets completos  
✅ Routers automáticos  
✅ Autenticación (Token, JWT)  
✅ Permisos personalizados  
✅ Filtros, búsqueda, ordenamiento  
✅ Paginación  
✅ Rate limiting

**Siguiente:** Autenticación Django profunda

---

## Recursos

- **[DRF Tutorial](https://www.django-rest-framework.org/tutorial/quickstart/)** - Tutorial oficial
- **[DRF Guide](https://www.django-rest-framework.org/)** - Guía completa
- **[Simple JWT](https://django-rest-framework-simplejwt.readthedocs.io/)** - JWT docs

DRF es el estándar para APIs Django. 🚀

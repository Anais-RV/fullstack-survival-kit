# Autenticación en Django

> **Sistema completo de usuarios, permisos y autenticación**

---

## Django User Model

### Modelo User incluido

Django incluye `django.contrib.auth.models.User`:

```python
from django.contrib.auth.models import User

# Campos:
# - username
# - password
# - email
# - first_name
# - last_name
# - is_staff
# - is_active
# - is_superuser
# - last_login
# - date_joined
```

### Crear usuarios

```python
from django.contrib.auth.models import User

# Crear usuario normal
usuario = User.objects.create_user(
    username='ana',
    email='ana@example.com',
    password='pass123'
)

# Crear superusuario
admin = User.objects.create_superuser(
    username='admin',
    email='admin@example.com',
    password='admin123'
)
```

### Autenticar usuario

```python
from django.contrib.auth import authenticate, login, logout

def vista_login(request):
    username = request.POST.get('username')
    password = request.POST.get('password')
    
    # Autenticar
    user = authenticate(request, username=username, password=password)
    
    if user is not None:
        # Login exitoso
        login(request, user)
        return redirect('home')
    else:
        # Credenciales inválidas
        return render(request, 'login.html', {
            'error': 'Credenciales inválidas'
        })

def vista_logout(request):
    logout(request)
    return redirect('login')
```

### Verificar autenticación

```python
def mi_vista(request):
    if request.user.is_authenticated:
        # Usuario autenticado
        usuario = request.user
        print(f'Hola {usuario.username}')
    else:
        # Usuario anónimo
        print('Usuario no autenticado')
```

---

## Custom User Model

**¡Recomendación:** Siempre usa Custom User desde el inicio!

### AbstractUser (extender User)

**models.py**

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class Usuario(AbstractUser):
    """Usuario personalizado"""
    # Mantiene todos los campos de User
    # Agregar campos adicionales
    telefono = models.CharField(max_length=20, blank=True)
    fecha_nacimiento = models.DateField(null=True, blank=True)
    biografia = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    
    def __str__(self):
        return self.username
```

**settings.py**

```python
AUTH_USER_MODEL = 'usuarios.Usuario'
```

**¡Migrar ANTES de crear usuarios!**

```bash
python manage.py makemigrations
python manage.py migrate
```

### AbstractBaseUser (control total)

```python
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
from django.db import models

class UsuarioManager(BaseUserManager):
    """Manager personalizado"""
    
    def create_user(self, email, password=None, **extra_fields):
        """Crear usuario normal"""
        if not email:
            raise ValueError('Email es obligatorio')
        
        email = self.normalize_email(email)
        usuario = self.model(email=email, **extra_fields)
        usuario.set_password(password)
        usuario.save(using=self._db)
        return usuario
    
    def create_superuser(self, email, password=None, **extra_fields):
        """Crear superusuario"""
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        
        return self.create_user(email, password, **extra_fields)

class Usuario(AbstractBaseUser, PermissionsMixin):
    """Usuario con email como username"""
    email = models.EmailField(unique=True)
    nombre = models.CharField(max_length=100)
    activo = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    creado_en = models.DateTimeField(auto_now_add=True)
    
    objects = UsuarioManager()
    
    USERNAME_FIELD = 'email'  # Campo para login
    REQUIRED_FIELDS = ['nombre']  # Campos requeridos en createsuperuser
    
    def __str__(self):
        return self.email
```

---

## Permisos y Grupos

### Permisos por defecto

Django crea automáticamente permisos para cada modelo:

```python
# usuarios.add_usuario
# usuarios.change_usuario
# usuarios.view_usuario
# usuarios.delete_usuario
```

### Verificar permisos

```python
def mi_vista(request):
    # Tiene permiso?
    if request.user.has_perm('usuarios.add_usuario'):
        # Puede crear usuarios
        pass
    
    # Múltiples permisos
    if request.user.has_perms(['usuarios.add_usuario', 'usuarios.change_usuario']):
        # Tiene ambos permisos
        pass
    
    # Es staff?
    if request.user.is_staff:
        pass
    
    # Es superuser?
    if request.user.is_superuser:
        pass
```

### Decoradores de permisos

```python
from django.contrib.auth.decorators import permission_required, login_required

@login_required
def perfil(request):
    """Requiere autenticación"""
    return render(request, 'perfil.html')

@permission_required('usuarios.add_usuario')
def crear_usuario(request):
    """Requiere permiso específico"""
    # ...
    pass

@permission_required(['usuarios.view_usuario', 'usuarios.change_usuario'])
def editar_usuario(request, id):
    """Requiere múltiples permisos"""
    # ...
    pass
```

### Grupos

```python
from django.contrib.auth.models import Group, Permission

# Crear grupo
editores = Group.objects.create(name='Editores')

# Agregar permisos al grupo
permiso_crear = Permission.objects.get(codename='add_post')
permiso_editar = Permission.objects.get(codename='change_post')

editores.permissions.add(permiso_crear, permiso_editar)

# Agregar usuario al grupo
usuario = User.objects.get(username='ana')
usuario.groups.add(editores)

# Verificar grupo
if usuario.groups.filter(name='Editores').exists():
    print('Es editor')
```

### Permisos personalizados

```python
class Post(models.Model):
    titulo = models.CharField(max_length=200)
    contenido = models.TextField()
    
    class Meta:
        permissions = [
            ('puede_publicar', 'Puede publicar posts'),
            ('puede_destacar', 'Puede destacar posts'),
        ]

# Uso
if usuario.has_perm('blog.puede_publicar'):
    # Puede publicar
    pass
```

---

## Autenticación con DRF

### Token Authentication

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

**Vista de login:**

```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import AllowAny
from rest_framework.response import Response
from rest_framework.authtoken.models import Token
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
            'username': user.username,
            'email': user.email
        })
    
    return Response({'error': 'Credenciales inválidas'}, status=400)

@api_view(['POST'])
def logout(request):
    """Eliminar token"""
    request.user.auth_token.delete()
    return Response({'mensaje': 'Sesión cerrada'})

@api_view(['GET'])
def perfil(request):
    """Obtener perfil del usuario autenticado"""
    return Response({
        'id': request.user.id,
        'username': request.user.username,
        'email': request.user.email
    })
```

**Uso del token:**

```bash
curl http://localhost:8000/api/perfil/ \
  -H "Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b"
```

---

## JWT con Simple JWT

### Instalación y configuración

```bash
pip install djangorestframework-simplejwt
```

**settings.py**

```python
from datetime import timedelta

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'UPDATE_LAST_LOGIN': True,
    
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,
    'AUTH_HEADER_TYPES': ('Bearer',),
}
```

**URLs:**

```python
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('api/token/verify/', TokenVerifyView.as_view(), name='token_verify'),
]
```

### JWT personalizado

```python
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer
from rest_framework_simplejwt.views import TokenObtainPairView

class CustomTokenObtainPairSerializer(TokenObtainPairSerializer):
    """Agregar datos adicionales al token"""
    
    @classmethod
    def get_token(cls, user):
        token = super().get_token(user)
        
        # Agregar claims personalizados
        token['username'] = user.username
        token['email'] = user.email
        
        return token
    
    def validate(self, attrs):
        """Personalizar respuesta"""
        data = super().validate(attrs)
        
        # Agregar datos adicionales
        data['user_id'] = self.user.id
        data['username'] = self.user.username
        data['email'] = self.user.email
        
        return data

class CustomTokenObtainPairView(TokenObtainPairView):
    serializer_class = CustomTokenObtainPairSerializer
```

### Vista de registro

```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import AllowAny
from rest_framework.response import Response
from rest_framework import status
from rest_framework_simplejwt.tokens import RefreshToken

@api_view(['POST'])
@permission_classes([AllowAny])
def registro(request):
    """Registrar nuevo usuario"""
    username = request.data.get('username')
    email = request.data.get('email')
    password = request.data.get('password')
    password_confirm = request.data.get('password_confirm')
    
    # Validaciones
    if User.objects.filter(username=username).exists():
        return Response({'error': 'Username ya existe'}, status=400)
    
    if User.objects.filter(email=email).exists():
        return Response({'error': 'Email ya existe'}, status=400)
    
    if password != password_confirm:
        return Response({'error': 'Las contraseñas no coinciden'}, status=400)
    
    # Crear usuario
    user = User.objects.create_user(
        username=username,
        email=email,
        password=password
    )
    
    # Generar tokens
    refresh = RefreshToken.for_user(user)
    
    return Response({
        'user_id': user.id,
        'username': user.username,
        'email': user.email,
        'access': str(refresh.access_token),
        'refresh': str(refresh)
    }, status=status.HTTP_201_CREATED)
```

---

## Cambiar contraseña

```python
@api_view(['POST'])
def cambiar_password(request):
    """Cambiar contraseña del usuario actual"""
    password_actual = request.data.get('password_actual')
    password_nueva = request.data.get('password_nueva')
    password_confirm = request.data.get('password_confirm')
    
    # Verificar password actual
    if not request.user.check_password(password_actual):
        return Response({'error': 'Contraseña actual incorrecta'}, status=400)
    
    # Verificar confirmación
    if password_nueva != password_confirm:
        return Response({'error': 'Las contraseñas no coinciden'}, status=400)
    
    # Cambiar password
    request.user.set_password(password_nueva)
    request.user.save()
    
    return Response({'mensaje': 'Contraseña actualizada'})
```

---

## Recuperar contraseña

```python
from django.core.mail import send_mail
from django.utils.crypto import get_random_string
from django.utils import timezone
from datetime import timedelta

class PasswordResetToken(models.Model):
    """Token para recuperar contraseña"""
    usuario = models.ForeignKey(User, on_delete=models.CASCADE)
    token = models.CharField(max_length=100, unique=True)
    creado_en = models.DateTimeField(auto_now_add=True)
    usado = models.BooleanField(default=False)
    
    def es_valido(self):
        """Token válido por 1 hora"""
        expira = self.creado_en + timedelta(hours=1)
        return not self.usado and timezone.now() < expira

@api_view(['POST'])
@permission_classes([AllowAny])
def solicitar_reset_password(request):
    """Enviar email con token"""
    email = request.data.get('email')
    
    try:
        user = User.objects.get(email=email)
    except User.DoesNotExist:
        # No revelar si el email existe
        return Response({'mensaje': 'Si el email existe, recibirás instrucciones'})
    
    # Crear token
    token = get_random_string(32)
    PasswordResetToken.objects.create(usuario=user, token=token)
    
    # Enviar email
    reset_url = f'http://tuapp.com/reset-password/{token}'
    send_mail(
        'Recuperar contraseña',
        f'Haz clic aquí para recuperar tu contraseña: {reset_url}',
        'noreply@tuapp.com',
        [email],
    )
    
    return Response({'mensaje': 'Si el email existe, recibirás instrucciones'})

@api_view(['POST'])
@permission_classes([AllowAny])
def reset_password(request, token):
    """Cambiar contraseña con token"""
    password = request.data.get('password')
    password_confirm = request.data.get('password_confirm')
    
    try:
        reset_token = PasswordResetToken.objects.get(token=token)
    except PasswordResetToken.DoesNotExist:
        return Response({'error': 'Token inválido'}, status=400)
    
    if not reset_token.es_valido():
        return Response({'error': 'Token expirado'}, status=400)
    
    if password != password_confirm:
        return Response({'error': 'Las contraseñas no coinciden'}, status=400)
    
    # Cambiar password
    user = reset_token.usuario
    user.set_password(password)
    user.save()
    
    # Marcar token como usado
    reset_token.usado = True
    reset_token.save()
    
    return Response({'mensaje': 'Contraseña actualizada'})
```

---

## Social Authentication

### django-allauth

```bash
pip install django-allauth
```

**settings.py**

```python
INSTALLED_APPS = [
    # ...
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
    'allauth.socialaccount.providers.github',
]

SITE_ID = 1

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
]

SOCIALACCOUNT_PROVIDERS = {
    'google': {
        'SCOPE': [
            'profile',
            'email',
        ],
        'AUTH_PARAMS': {
            'access_type': 'online',
        }
    }
}
```

---

## Ejemplo completo: Sistema de auth

**serializers.py**

```python
from rest_framework import serializers
from django.contrib.auth.models import User

class UsuarioSerializer(serializers.ModelSerializer):
    """Serializer para usuario"""
    password = serializers.CharField(write_only=True)
    
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'password', 'first_name', 'last_name']
        extra_kwargs = {
            'password': {'write_only': True}
        }
    
    def create(self, validated_data):
        """Crear usuario con password hasheada"""
        password = validated_data.pop('password')
        user = User.objects.create(**validated_data)
        user.set_password(password)
        user.save()
        return user

class LoginSerializer(serializers.Serializer):
    """Serializer para login"""
    username = serializers.CharField()
    password = serializers.CharField()

class CambiarPasswordSerializer(serializers.Serializer):
    """Serializer para cambiar password"""
    password_actual = serializers.CharField()
    password_nueva = serializers.CharField()
    password_confirm = serializers.CharField()
    
    def validate(self, data):
        if data['password_nueva'] != data['password_confirm']:
            raise serializers.ValidationError('Las contraseñas no coinciden')
        return data
```

**views.py**

```python
from rest_framework import viewsets, status
from rest_framework.decorators import action, api_view, permission_classes
from rest_framework.permissions import AllowAny, IsAuthenticated
from rest_framework.response import Response
from rest_framework_simplejwt.tokens import RefreshToken
from django.contrib.auth import authenticate
from django.contrib.auth.models import User

from .serializers import UsuarioSerializer, LoginSerializer, CambiarPasswordSerializer

class AuthViewSet(viewsets.GenericViewSet):
    """ViewSet para autenticación"""
    permission_classes = [AllowAny]
    
    @action(detail=False, methods=['post'])
    def registro(self, request):
        """Registrar nuevo usuario"""
        serializer = UsuarioSerializer(data=request.data)
        
        if serializer.is_valid():
            user = serializer.save()
            
            # Generar tokens
            refresh = RefreshToken.for_user(user)
            
            return Response({
                'user': UsuarioSerializer(user).data,
                'access': str(refresh.access_token),
                'refresh': str(refresh)
            }, status=status.HTTP_201_CREATED)
        
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    @action(detail=False, methods=['post'])
    def login(self, request):
        """Login de usuario"""
        serializer = LoginSerializer(data=request.data)
        
        if serializer.is_valid():
            user = authenticate(
                username=serializer.validated_data['username'],
                password=serializer.validated_data['password']
            )
            
            if user:
                refresh = RefreshToken.for_user(user)
                
                return Response({
                    'user': UsuarioSerializer(user).data,
                    'access': str(refresh.access_token),
                    'refresh': str(refresh)
                })
            
            return Response({'error': 'Credenciales inválidas'}, status=400)
        
        return Response(serializer.errors, status=400)

class UsuarioViewSet(viewsets.ModelViewSet):
    """ViewSet para usuarios"""
    queryset = User.objects.all()
    serializer_class = UsuarioSerializer
    permission_classes = [IsAuthenticated]
    
    @action(detail=False, methods=['get'])
    def me(self, request):
        """Obtener usuario actual"""
        serializer = self.get_serializer(request.user)
        return Response(serializer.data)
    
    @action(detail=False, methods=['post'])
    def cambiar_password(self, request):
        """Cambiar contraseña"""
        serializer = CambiarPasswordSerializer(data=request.data)
        
        if serializer.is_valid():
            if not request.user.check_password(serializer.validated_data['password_actual']):
                return Response({'error': 'Contraseña actual incorrecta'}, status=400)
            
            request.user.set_password(serializer.validated_data['password_nueva'])
            request.user.save()
            
            return Response({'mensaje': 'Contraseña actualizada'})
        
        return Response(serializer.errors, status=400)
```

**urls.py**

```python
from rest_framework.routers import DefaultRouter
from rest_framework_simplejwt.views import TokenRefreshView

router = DefaultRouter()
router.register('auth', AuthViewSet, basename='auth')
router.register('usuarios', UsuarioViewSet)

urlpatterns = [
    path('api/token/refresh/', TokenRefreshView.as_view()),
] + router.urls
```

---

## Resumen

Has aprendido:

✅ Django User Model  
✅ Custom User Model (AbstractUser, AbstractBaseUser)  
✅ Permisos y grupos  
✅ Token Authentication  
✅ JWT con Simple JWT  
✅ Registro y login  
✅ Cambiar y recuperar contraseña

**Siguiente:** Django Admin

---

## Recursos

- **[Authentication](https://docs.djangoproject.com/en/5.0/topics/auth/)** - Auth de Django
- **[Simple JWT](https://django-rest-framework-simplejwt.readthedocs.io/)** - JWT docs
- **[django-allauth](https://django-allauth.readthedocs.io/)** - Social auth

El sistema de auth de Django es completo. 🔐

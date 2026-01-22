# Testing Backend (Pytest + Django)

> **Testear modelos, serializers, views y API endpoints de Django**

---

## Configuración inicial

### Instalar pytest

```bash
pip install pytest pytest-django pytest-cov factory-boy
```

### pytest.ini

```ini
# pytest.ini
[pytest]
DJANGO_SETTINGS_MODULE = core.settings
python_files = tests.py test_*.py *_tests.py
addopts = --reuse-db --nomigrations
```

### conftest.py

```python
# conftest.py (raíz del proyecto)
import pytest
from django.contrib.auth import get_user_model

User = get_user_model()

@pytest.fixture
def user():
    return User.objects.create_user(
        username='testuser',
        email='test@example.com',
        password='testpass123'
    )

@pytest.fixture
def api_client():
    from rest_framework.test import APIClient
    return APIClient()

@pytest.fixture
def authenticated_client(api_client, user):
    api_client.force_authenticate(user=user)
    return api_client
```

---

## Tests de modelos

### Modelo básico

```python
# productos/models.py
from django.db import models
from django.core.exceptions import ValidationError

class Producto(models.Model):
    nombre = models.CharField(max_length=200)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField(default=0)
    activo = models.BooleanField(default=True)
    
    def clean(self):
        if self.precio <= 0:
            raise ValidationError('El precio debe ser positivo')
        if self.stock < 0:
            raise ValidationError('El stock no puede ser negativo')
    
    def esta_disponible(self):
        return self.activo and self.stock > 0
    
    def __str__(self):
        return self.nombre
```

```python
# productos/tests/test_models.py
import pytest
from django.core.exceptions import ValidationError
from productos.models import Producto

@pytest.mark.django_db
class TestProductoModel:
    def test_crear_producto(self):
        producto = Producto.objects.create(
            nombre='Laptop',
            precio=1200,
            stock=10
        )
        assert producto.nombre == 'Laptop'
        assert producto.precio == 1200
        assert producto.stock == 10
        assert producto.activo is True
    
    def test_producto_str(self):
        producto = Producto.objects.create(nombre='Mouse', precio=25)
        assert str(producto) == 'Mouse'
    
    def test_producto_disponible(self):
        producto = Producto.objects.create(
            nombre='Laptop',
            precio=1200,
            stock=10,
            activo=True
        )
        assert producto.esta_disponible() is True
    
    def test_producto_no_disponible_sin_stock(self):
        producto = Producto.objects.create(
            nombre='Laptop',
            precio=1200,
            stock=0,
            activo=True
        )
        assert producto.esta_disponible() is False
    
    def test_producto_no_disponible_inactivo(self):
        producto = Producto.objects.create(
            nombre='Laptop',
            precio=1200,
            stock=10,
            activo=False
        )
        assert producto.esta_disponible() is False
    
    def test_precio_positivo(self):
        producto = Producto(nombre='Laptop', precio=-100, stock=10)
        with pytest.raises(ValidationError):
            producto.full_clean()
    
    def test_stock_no_negativo(self):
        producto = Producto(nombre='Laptop', precio=1200, stock=-5)
        with pytest.raises(ValidationError):
            producto.full_clean()
```

---

## Tests de serializers

```python
# productos/serializers.py
from rest_framework import serializers
from .models import Producto

class ProductoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Producto
        fields = ['id', 'nombre', 'precio', 'stock', 'activo']
    
    def validate_precio(self, value):
        if value <= 0:
            raise serializers.ValidationError("El precio debe ser positivo")
        return value
    
    def validate_stock(self, value):
        if value < 0:
            raise serializers.ValidationError("El stock no puede ser negativo")
        return value
```

```python
# productos/tests/test_serializers.py
import pytest
from productos.models import Producto
from productos.serializers import ProductoSerializer

@pytest.mark.django_db
class TestProductoSerializer:
    def test_serializar_producto(self):
        producto = Producto.objects.create(
            nombre='Laptop',
            precio=1200,
            stock=10
        )
        serializer = ProductoSerializer(producto)
        data = serializer.data
        
        assert data['nombre'] == 'Laptop'
        assert float(data['precio']) == 1200.0
        assert data['stock'] == 10
        assert data['activo'] is True
    
    def test_deserializar_datos_validos(self):
        data = {
            'nombre': 'Mouse',
            'precio': 25.99,
            'stock': 50
        }
        serializer = ProductoSerializer(data=data)
        assert serializer.is_valid()
        producto = serializer.save()
        
        assert producto.nombre == 'Mouse'
        assert producto.precio == 25.99
        assert producto.stock == 50
    
    def test_validar_precio_positivo(self):
        data = {
            'nombre': 'Laptop',
            'precio': -100,
            'stock': 10
        }
        serializer = ProductoSerializer(data=data)
        assert not serializer.is_valid()
        assert 'precio' in serializer.errors
    
    def test_validar_stock_no_negativo(self):
        data = {
            'nombre': 'Laptop',
            'precio': 1200,
            'stock': -5
        }
        serializer = ProductoSerializer(data=data)
        assert not serializer.is_valid()
        assert 'stock' in serializer.errors
    
    def test_campos_requeridos(self):
        data = {}
        serializer = ProductoSerializer(data=data)
        assert not serializer.is_valid()
        assert 'nombre' in serializer.errors
        assert 'precio' in serializer.errors
```

---

## Tests de ViewSets y APIs

```python
# productos/tests/test_views.py
import pytest
from django.urls import reverse
from rest_framework import status
from productos.models import Producto

@pytest.mark.django_db
class TestProductoAPI:
    def test_listar_productos(self, api_client):
        # Crear productos
        Producto.objects.create(nombre='Laptop', precio=1200, stock=10)
        Producto.objects.create(nombre='Mouse', precio=25, stock=50)
        
        # Request
        url = reverse('producto-list')
        response = api_client.get(url)
        
        # Assert
        assert response.status_code == status.HTTP_200_OK
        assert len(response.data) == 2
    
    def test_obtener_producto_por_id(self, api_client):
        producto = Producto.objects.create(nombre='Laptop', precio=1200)
        
        url = reverse('producto-detail', kwargs={'pk': producto.id})
        response = api_client.get(url)
        
        assert response.status_code == status.HTTP_200_OK
        assert response.data['nombre'] == 'Laptop'
        assert float(response.data['precio']) == 1200.0
    
    def test_crear_producto_autenticado(self, authenticated_client):
        data = {
            'nombre': 'Teclado',
            'precio': 89.99,
            'stock': 30
        }
        
        url = reverse('producto-list')
        response = authenticated_client.post(url, data, format='json')
        
        assert response.status_code == status.HTTP_201_CREATED
        assert response.data['nombre'] == 'Teclado'
        assert Producto.objects.count() == 1
    
    def test_crear_producto_sin_autenticar(self, api_client):
        data = {
            'nombre': 'Teclado',
            'precio': 89.99,
            'stock': 30
        }
        
        url = reverse('producto-list')
        response = api_client.post(url, data, format='json')
        
        assert response.status_code == status.HTTP_401_UNAUTHORIZED
    
    def test_actualizar_producto(self, authenticated_client):
        producto = Producto.objects.create(nombre='Laptop', precio=1200)
        
        data = {
            'nombre': 'Laptop Pro',
            'precio': 1500,
            'stock': 5
        }
        
        url = reverse('producto-detail', kwargs={'pk': producto.id})
        response = authenticated_client.put(url, data, format='json')
        
        assert response.status_code == status.HTTP_200_OK
        assert response.data['nombre'] == 'Laptop Pro'
        
        producto.refresh_from_db()
        assert producto.nombre == 'Laptop Pro'
        assert producto.precio == 1500
    
    def test_eliminar_producto(self, authenticated_client):
        producto = Producto.objects.create(nombre='Laptop', precio=1200)
        
        url = reverse('producto-detail', kwargs={'pk': producto.id})
        response = authenticated_client.delete(url)
        
        assert response.status_code == status.HTTP_204_NO_CONTENT
        assert Producto.objects.count() == 0
    
    def test_producto_no_encontrado(self, api_client):
        url = reverse('producto-detail', kwargs={'pk': 9999})
        response = api_client.get(url)
        assert response.status_code == status.HTTP_404_NOT_FOUND
```

---

## Factory Boy (datos de prueba)

### Crear factory

```python
# productos/tests/factories.py
import factory
from factory.django import DjangoModelFactory
from productos.models import Producto
from django.contrib.auth import get_user_model

User = get_user_model()

class UserFactory(DjangoModelFactory):
    class Meta:
        model = User
    
    username = factory.Sequence(lambda n: f'user{n}')
    email = factory.LazyAttribute(lambda obj: f'{obj.username}@example.com')
    password = factory.PostGenerationMethodCall('set_password', 'password123')

class ProductoFactory(DjangoModelFactory):
    class Meta:
        model = Producto
    
    nombre = factory.Faker('word')
    precio = factory.Faker('pydecimal', left_digits=4, right_digits=2, positive=True)
    stock = factory.Faker('random_int', min=0, max=100)
    activo = True
```

### Usar factories en tests

```python
# productos/tests/test_views.py
import pytest
from productos.tests.factories import ProductoFactory, UserFactory

@pytest.mark.django_db
class TestProductoAPIWithFactories:
    def test_listar_productos_con_factory(self, api_client):
        # Crear 5 productos
        ProductoFactory.create_batch(5)
        
        url = reverse('producto-list')
        response = api_client.get(url)
        
        assert response.status_code == 200
        assert len(response.data) == 5
    
    def test_crear_producto_con_factory(self, api_client):
        user = UserFactory()
        api_client.force_authenticate(user=user)
        
        data = {
            'nombre': 'Laptop',
            'precio': 1200,
            'stock': 10
        }
        
        url = reverse('producto-list')
        response = api_client.post(url, data, format='json')
        
        assert response.status_code == 201
```

---

## Tests de autenticación JWT

```python
# usuarios/tests/test_auth.py
import pytest
from django.urls import reverse
from rest_framework import status
from django.contrib.auth import get_user_model

User = get_user_model()

@pytest.mark.django_db
class TestAuthAPI:
    def test_register(self, api_client):
        data = {
            'username': 'newuser',
            'email': 'newuser@example.com',
            'password': 'password123',
            'password2': 'password123'
        }
        
        url = reverse('register')
        response = api_client.post(url, data, format='json')
        
        assert response.status_code == status.HTTP_201_CREATED
        assert 'tokens' in response.data
        assert 'access' in response.data['tokens']
        assert 'refresh' in response.data['tokens']
        assert User.objects.filter(username='newuser').exists()
    
    def test_register_passwords_no_coinciden(self, api_client):
        data = {
            'username': 'newuser',
            'email': 'newuser@example.com',
            'password': 'password123',
            'password2': 'password456'
        }
        
        url = reverse('register')
        response = api_client.post(url, data, format='json')
        
        assert response.status_code == status.HTTP_400_BAD_REQUEST
    
    def test_login(self, api_client):
        # Crear usuario
        user = User.objects.create_user(
            username='testuser',
            password='password123'
        )
        
        data = {
            'username': 'testuser',
            'password': 'password123'
        }
        
        url = reverse('token_obtain_pair')
        response = api_client.post(url, data, format='json')
        
        assert response.status_code == status.HTTP_200_OK
        assert 'access' in response.data
        assert 'refresh' in response.data
    
    def test_login_credenciales_incorrectas(self, api_client):
        User.objects.create_user(
            username='testuser',
            password='password123'
        )
        
        data = {
            'username': 'testuser',
            'password': 'wrong_password'
        }
        
        url = reverse('token_obtain_pair')
        response = api_client.post(url, data, format='json')
        
        assert response.status_code == status.HTTP_401_UNAUTHORIZED
    
    def test_refresh_token(self, api_client):
        user = User.objects.create_user(
            username='testuser',
            password='password123'
        )
        
        # Obtener tokens
        login_url = reverse('token_obtain_pair')
        login_response = api_client.post(login_url, {
            'username': 'testuser',
            'password': 'password123'
        }, format='json')
        
        refresh_token = login_response.data['refresh']
        
        # Refresh
        refresh_url = reverse('token_refresh')
        refresh_response = api_client.post(refresh_url, {
            'refresh': refresh_token
        }, format='json')
        
        assert refresh_response.status_code == status.HTTP_200_OK
        assert 'access' in refresh_response.data
    
    def test_profile_autenticado(self, authenticated_client, user):
        url = reverse('profile')
        response = authenticated_client.get(url)
        
        assert response.status_code == status.HTTP_200_OK
        assert response.data['username'] == user.username
    
    def test_profile_sin_autenticar(self, api_client):
        url = reverse('profile')
        response = api_client.get(url)
        
        assert response.status_code == status.HTTP_401_UNAUTHORIZED
```

---

## Tests de permisos

```python
# productos/tests/test_permissions.py
import pytest
from django.urls import reverse
from rest_framework import status
from productos.models import Producto
from productos.tests.factories import UserFactory, ProductoFactory

@pytest.mark.django_db
class TestProductoPermissions:
    def test_listar_sin_autenticar(self, api_client):
        ProductoFactory.create_batch(3)
        
        url = reverse('producto-list')
        response = api_client.get(url)
        
        # Asumiendo IsAuthenticatedOrReadOnly
        assert response.status_code == status.HTTP_200_OK
    
    def test_crear_sin_autenticar(self, api_client):
        data = {
            'nombre': 'Laptop',
            'precio': 1200,
            'stock': 10
        }
        
        url = reverse('producto-list')
        response = api_client.post(url, data, format='json')
        
        assert response.status_code == status.HTTP_401_UNAUTHORIZED
    
    def test_crear_autenticado(self, authenticated_client):
        data = {
            'nombre': 'Laptop',
            'precio': 1200,
            'stock': 10
        }
        
        url = reverse('producto-list')
        response = authenticated_client.post(url, data, format='json')
        
        assert response.status_code == status.HTTP_201_CREATED
```

---

## Coverage

```bash
# Ejecutar tests con coverage
pytest --cov=productos --cov-report=html

# Ver reporte
open htmlcov/index.html
```

---

## Fixtures reutilizables

```python
# conftest.py
import pytest
from productos.tests.factories import ProductoFactory, UserFactory

@pytest.fixture
def producto():
    return ProductoFactory()

@pytest.fixture
def productos_lista():
    return ProductoFactory.create_batch(10)

@pytest.fixture
def admin_user():
    user = UserFactory(username='admin', is_staff=True, is_superuser=True)
    return user

@pytest.fixture
def admin_client(api_client, admin_user):
    api_client.force_authenticate(user=admin_user)
    return api_client
```

**Uso:**
```python
def test_listar_productos(api_client, productos_lista):
    url = reverse('producto-list')
    response = api_client.get(url)
    assert len(response.data) == 10
```

---

## Tests parametrizados

```python
@pytest.mark.parametrize('precio,valido', [
    (100, True),
    (0.01, True),
    (0, False),
    (-10, False),
])
def test_validar_precio(precio, valido):
    data = {
        'nombre': 'Producto',
        'precio': precio,
        'stock': 10
    }
    serializer = ProductoSerializer(data=data)
    assert serializer.is_valid() == valido
```

---

## Resumen

**Setup:**
- pytest + pytest-django
- Factory Boy para datos de prueba
- Fixtures reutilizables

**Tests de:**
- Modelos (métodos, validaciones)
- Serializers (validación, transformación)
- Views/APIs (endpoints, permisos, status codes)
- Autenticación (registro, login, tokens)

**Mejores prácticas:**
- Usar factories en lugar de crear objetos manualmente
- Fixtures para setup común
- Tests parametrizados para múltiples casos
- Coverage > 80%

**Próximo módulo:** Tests de integración frontend + backend

---

## Recursos

- **[Pytest](https://pytest.org/)** - Documentación oficial
- **[pytest-django](https://pytest-django.readthedocs.io/)** - Plugin Django
- **[Factory Boy](https://factoryboy.readthedocs.io/)** - Crear datos de prueba
- **[DRF Testing](https://www.django-rest-framework.org/api-guide/testing/)** - Testing en DRF

¡Backend testeado! 🧪

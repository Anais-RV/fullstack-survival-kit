# Testing en Django

> **Tests automatizados para código confiable**

---

## ¿Por qué testear?

**Sin tests:**
```python
# ¿Funciona? 🤷
# ¿Qué puede romper? 🤷
# ¿Refactorizar? 😰
```

**Con tests:**
```python
# ✅ Funciona
# ✅ Sé qué puede romper
# ✅ Refactorizo con confianza
```

---

## Django TestCase

### Test básico

```python
from django.test import TestCase
from .models import Usuario

class UsuarioTestCase(TestCase):
    """Tests del modelo Usuario"""
    
    def setUp(self):
        """Ejecutar antes de cada test"""
        self.usuario = Usuario.objects.create(
            nombre='Ana García',
            email='ana@example.com'
        )
    
    def test_usuario_creation(self):
        """Verificar que el usuario se creó"""
        self.assertEqual(self.usuario.nombre, 'Ana García')
        self.assertEqual(self.usuario.email, 'ana@example.com')
        self.assertTrue(self.usuario.activo)
    
    def test_usuario_str(self):
        """Verificar __str__"""
        self.assertEqual(str(self.usuario), 'Ana García (ana@example.com)')
```

**Ejecutar tests:**

```bash
# Todos los tests
python manage.py test

# Una app
python manage.py test usuarios

# Un archivo
python manage.py test usuarios.tests.test_models

# Una clase
python manage.py test usuarios.tests.test_models.UsuarioTestCase

# Un método
python manage.py test usuarios.tests.test_models.UsuarioTestCase.test_usuario_creation
```

---

## Assertions

```python
class MiTestCase(TestCase):
    def test_assertions(self):
        # Igualdad
        self.assertEqual(1 + 1, 2)
        self.assertNotEqual(1, 2)
        
        # Booleanos
        self.assertTrue(True)
        self.assertFalse(False)
        
        # None
        self.assertIsNone(None)
        self.assertIsNotNone('algo')
        
        # Contiene
        self.assertIn('a', 'hola')
        self.assertNotIn('x', 'hola')
        
        # Listas/Sets
        self.assertListEqual([1, 2], [1, 2])
        self.assertCountEqual([1, 2, 3], [3, 2, 1])  # Orden no importa
        
        # Excepciones
        with self.assertRaises(ValueError):
            int('abc')
        
        # Queries
        self.assertQuerysetEqual(
            Usuario.objects.filter(activo=True),
            ['<Usuario: Ana>']
        )
```

---

## Testing Models

```python
from django.test import TestCase
from django.core.exceptions import ValidationError
from .models import Producto

class ProductoTestCase(TestCase):
    """Tests del modelo Producto"""
    
    def setUp(self):
        """Crear producto de prueba"""
        self.producto = Producto.objects.create(
            nombre='Laptop',
            precio=1000,
            stock=10
        )
    
    def test_producto_creation(self):
        """Verificar creación"""
        self.assertEqual(self.producto.nombre, 'Laptop')
        self.assertEqual(self.producto.precio, 1000)
        self.assertEqual(self.producto.stock, 10)
        self.assertTrue(self.producto.activo)
    
    def test_tiene_stock(self):
        """Test del método tiene_stock"""
        self.assertTrue(self.producto.tiene_stock())
        
        self.producto.stock = 0
        self.assertFalse(self.producto.tiene_stock())
    
    def test_reducir_stock(self):
        """Test de reducir stock"""
        self.producto.reducir_stock(5)
        self.assertEqual(self.producto.stock, 5)
    
    def test_reducir_stock_insuficiente(self):
        """Test de error con stock insuficiente"""
        with self.assertRaises(ValueError) as context:
            self.producto.reducir_stock(20)
        
        self.assertIn('insuficiente', str(context.exception).lower())
    
    def test_precio_negativo(self):
        """No permitir precio negativo"""
        producto = Producto(nombre='Test', precio=-100, stock=0)
        
        with self.assertRaises(ValidationError):
            producto.full_clean()
    
    def test_str_representation(self):
        """Test de __str__"""
        self.assertEqual(str(self.producto), 'Laptop')
```

---

## Testing Views (FBV)

```python
from django.test import TestCase, Client
from django.urls import reverse
from .models import Usuario

class UsuarioViewTestCase(TestCase):
    """Tests de vistas de Usuario"""
    
    def setUp(self):
        """Setup para cada test"""
        self.client = Client()
        self.usuario = Usuario.objects.create(
            nombre='Ana',
            email='ana@example.com'
        )
    
    def test_usuario_list_view(self):
        """Test de GET /usuarios/"""
        response = self.client.get(reverse('usuario_list'))
        
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Ana')
    
    def test_usuario_detail_view(self):
        """Test de GET /usuarios/<id>/"""
        url = reverse('usuario_detail', kwargs={'id': self.usuario.id})
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'ana@example.com')
    
    def test_usuario_create_view(self):
        """Test de POST /usuarios/"""
        data = {
            'nombre': 'Bob',
            'email': 'bob@example.com'
        }
        
        response = self.client.post(reverse('usuario_create'), data)
        
        self.assertEqual(response.status_code, 201)
        self.assertTrue(Usuario.objects.filter(email='bob@example.com').exists())
    
    def test_usuario_update_view(self):
        """Test de PUT /usuarios/<id>/"""
        url = reverse('usuario_update', kwargs={'id': self.usuario.id})
        data = {
            'nombre': 'Ana María',
            'email': 'ana@example.com'
        }
        
        response = self.client.put(url, data, content_type='application/json')
        
        self.assertEqual(response.status_code, 200)
        
        self.usuario.refresh_from_db()
        self.assertEqual(self.usuario.nombre, 'Ana María')
    
    def test_usuario_delete_view(self):
        """Test de DELETE /usuarios/<id>/"""
        url = reverse('usuario_delete', kwargs={'id': self.usuario.id})
        response = self.client.delete(url)
        
        self.assertEqual(response.status_code, 204)
        self.assertFalse(Usuario.objects.filter(id=self.usuario.id).exists())
    
    def test_usuario_not_found(self):
        """Test de usuario no encontrado"""
        url = reverse('usuario_detail', kwargs={'id': 9999})
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, 404)
```

---

## Testing con DRF (APITestCase)

```python
from rest_framework.test import APITestCase, APIClient
from rest_framework import status
from django.urls import reverse
from django.contrib.auth.models import User
from .models import Post

class PostAPITestCase(APITestCase):
    """Tests de la API de Posts"""
    
    def setUp(self):
        """Setup para cada test"""
        self.client = APIClient()
        
        # Crear usuario
        self.user = User.objects.create_user(
            username='ana',
            password='pass123'
        )
        
        # Crear post
        self.post = Post.objects.create(
            titulo='Mi post',
            contenido='Contenido del post',
            autor=self.user
        )
    
    def test_list_posts_sin_auth(self):
        """Test de GET /posts/ sin autenticación"""
        url = reverse('post-list')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
    
    def test_create_post_sin_auth(self):
        """Test de POST /posts/ sin autenticación"""
        url = reverse('post-list')
        data = {'titulo': 'Nuevo post', 'contenido': 'Contenido'}
        
        response = self.client.post(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    def test_create_post_con_auth(self):
        """Test de POST /posts/ con autenticación"""
        self.client.force_authenticate(user=self.user)
        
        url = reverse('post-list')
        data = {
            'titulo': 'Nuevo post',
            'contenido': 'Contenido del nuevo post'
        }
        
        response = self.client.post(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data['titulo'], 'Nuevo post')
        self.assertEqual(Post.objects.count(), 2)
    
    def test_update_post_propio(self):
        """Test de PUT /posts/<id>/ del propio post"""
        self.client.force_authenticate(user=self.user)
        
        url = reverse('post-detail', kwargs={'pk': self.post.pk})
        data = {
            'titulo': 'Post actualizado',
            'contenido': 'Contenido actualizado'
        }
        
        response = self.client.put(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        self.post.refresh_from_db()
        self.assertEqual(self.post.titulo, 'Post actualizado')
    
    def test_update_post_ajeno(self):
        """Test de PUT /posts/<id>/ de post ajeno"""
        # Crear otro usuario
        otro_user = User.objects.create_user(username='bob', password='pass123')
        self.client.force_authenticate(user=otro_user)
        
        url = reverse('post-detail', kwargs={'pk': self.post.pk})
        data = {'titulo': 'Intento actualizar', 'contenido': 'No debería poder'}
        
        response = self.client.put(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
    
    def test_delete_post(self):
        """Test de DELETE /posts/<id>/"""
        self.client.force_authenticate(user=self.user)
        
        url = reverse('post-detail', kwargs={'pk': self.post.pk})
        response = self.client.delete(url)
        
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        self.assertEqual(Post.objects.count(), 0)
    
    def test_filtrar_posts_por_autor(self):
        """Test de filtrado"""
        url = reverse('post-list')
        response = self.client.get(url, {'autor': self.user.id})
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
    
    def test_buscar_posts(self):
        """Test de búsqueda"""
        url = reverse('post-list')
        response = self.client.get(url, {'search': 'post'})
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertGreater(len(response.data['results']), 0)
```

---

## Testing Autenticación

```python
from rest_framework.test import APITestCase
from rest_framework import status
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

class AuthAPITestCase(APITestCase):
    """Tests de autenticación"""
    
    def setUp(self):
        """Setup"""
        self.user = User.objects.create_user(
            username='ana',
            email='ana@example.com',
            password='pass123'
        )
    
    def test_registro(self):
        """Test de registro de usuario"""
        url = '/api/auth/registro/'
        data = {
            'username': 'bob',
            'email': 'bob@example.com',
            'password': 'pass123',
            'password_confirm': 'pass123'
        }
        
        response = self.client.post(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertIn('access', response.data)
        self.assertIn('refresh', response.data)
        self.assertTrue(User.objects.filter(username='bob').exists())
    
    def test_registro_password_no_coincide(self):
        """Test de registro con passwords distintas"""
        url = '/api/auth/registro/'
        data = {
            'username': 'bob',
            'email': 'bob@example.com',
            'password': 'pass123',
            'password_confirm': 'pass456'
        }
        
        response = self.client.post(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
    def test_login_exitoso(self):
        """Test de login exitoso"""
        url = '/api/auth/login/'
        data = {
            'username': 'ana',
            'password': 'pass123'
        }
        
        response = self.client.post(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIn('access', response.data)
        self.assertIn('refresh', response.data)
    
    def test_login_fallido(self):
        """Test de login con credenciales incorrectas"""
        url = '/api/auth/login/'
        data = {
            'username': 'ana',
            'password': 'wrong_password'
        }
        
        response = self.client.post(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
    def test_perfil_sin_auth(self):
        """Test de acceder al perfil sin autenticación"""
        url = '/api/usuarios/me/'
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    def test_perfil_con_auth(self):
        """Test de acceder al perfil autenticado"""
        self.client.force_authenticate(user=self.user)
        
        url = '/api/usuarios/me/'
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['username'], 'ana')
    
    def test_cambiar_password(self):
        """Test de cambiar contraseña"""
        self.client.force_authenticate(user=self.user)
        
        url = '/api/usuarios/cambiar-password/'
        data = {
            'password_actual': 'pass123',
            'password_nueva': 'newpass123',
            'password_confirm': 'newpass123'
        }
        
        response = self.client.post(url, data)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Verificar que la nueva password funciona
        self.user.refresh_from_db()
        self.assertTrue(self.user.check_password('newpass123'))
```

---

## Fixtures

### Crear fixtures

```python
# tests/fixtures.py
from django.contrib.auth.models import User
from .models import Categoria, Producto

def crear_usuario(**kwargs):
    """Factory para crear usuarios"""
    defaults = {
        'username': 'testuser',
        'email': 'test@example.com',
        'password': 'pass123'
    }
    defaults.update(kwargs)
    
    return User.objects.create_user(**defaults)

def crear_categoria(**kwargs):
    """Factory para crear categorías"""
    defaults = {
        'nombre': 'Test Categoría',
        'slug': 'test-categoria'
    }
    defaults.update(kwargs)
    
    return Categoria.objects.create(**defaults)

def crear_producto(**kwargs):
    """Factory para crear productos"""
    defaults = {
        'nombre': 'Test Producto',
        'slug': 'test-producto',
        'precio': 100,
        'stock': 10
    }
    
    if 'categoria' not in kwargs:
        kwargs['categoria'] = crear_categoria()
    
    defaults.update(kwargs)
    
    return Producto.objects.create(**defaults)
```

### Usar fixtures

```python
from django.test import TestCase
from .tests.fixtures import crear_usuario, crear_producto

class ProductoTestCase(TestCase):
    def setUp(self):
        self.usuario = crear_usuario()
        self.producto = crear_producto(
            nombre='Laptop',
            precio=1000
        )
    
    def test_producto(self):
        self.assertEqual(self.producto.nombre, 'Laptop')
        self.assertEqual(self.producto.precio, 1000)
```

---

## Mocking

```python
from unittest.mock import patch, Mock
from django.test import TestCase
from .services import EmailService

class EmailServiceTestCase(TestCase):
    """Tests de servicio de email"""
    
    @patch('django.core.mail.send_mail')
    def test_enviar_email(self, mock_send_mail):
        """Test de enviar email (mock)"""
        mock_send_mail.return_value = 1  # Simular envío exitoso
        
        service = EmailService()
        resultado = service.enviar_bienvenida('ana@example.com')
        
        self.assertTrue(resultado)
        
        # Verificar que se llamó con los argumentos correctos
        mock_send_mail.assert_called_once()
        args, kwargs = mock_send_mail.call_args
        self.assertEqual(kwargs['recipient_list'], ['ana@example.com'])
    
    @patch('requests.get')
    def test_api_externa(self, mock_get):
        """Test de llamada a API externa"""
        # Simular respuesta
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = {'data': 'test'}
        mock_get.return_value = mock_response
        
        # Tu código que llama a la API
        from .services import APIService
        service = APIService()
        resultado = service.obtener_datos()
        
        self.assertEqual(resultado, {'data': 'test'})
        mock_get.assert_called_once()
```

---

## Coverage (cobertura de tests)

**Instalación:**

```bash
pip install coverage
```

**Ejecutar con coverage:**

```bash
# Ejecutar tests con coverage
coverage run --source='.' manage.py test

# Ver reporte en terminal
coverage report

# Generar reporte HTML
coverage html

# Abrir htmlcov/index.html en navegador
```

**Output:**

```
Name                      Stmts   Miss  Cover
---------------------------------------------
usuarios/models.py           45      2    96%
usuarios/views.py            78      5    94%
usuarios/serializers.py      32      0   100%
---------------------------------------------
TOTAL                       155      7    95%
```

---

## pytest-django

**Alternativa a TestCase:**

```bash
pip install pytest pytest-django
```

**pytest.ini:**

```ini
[pytest]
DJANGO_SETTINGS_MODULE = miproyecto.settings
python_files = tests.py test_*.py *_tests.py
```

**Tests con pytest:**

```python
import pytest
from django.contrib.auth.models import User

@pytest.mark.django_db
def test_crear_usuario():
    """Test con pytest"""
    user = User.objects.create_user(username='ana', password='pass123')
    assert user.username == 'ana'
    assert user.check_password('pass123')

@pytest.mark.django_db
def test_usuario_count():
    """Test de count"""
    User.objects.create_user(username='ana', password='pass123')
    User.objects.create_user(username='bob', password='pass123')
    
    assert User.objects.count() == 2
```

**Fixtures de pytest:**

```python
import pytest
from django.contrib.auth.models import User

@pytest.fixture
def usuario():
    """Fixture de usuario"""
    return User.objects.create_user(username='ana', password='pass123')

@pytest.mark.django_db
def test_usuario_auth(usuario):
    """Test usando fixture"""
    assert usuario.is_authenticated
    assert usuario.username == 'ana'
```

---

## Best Practices

### 1. Arrange-Act-Assert

```python
def test_crear_pedido(self):
    # Arrange (preparar)
    usuario = crear_usuario()
    producto = crear_producto(precio=100)
    
    # Act (ejecutar)
    pedido = Pedido.objects.create(usuario=usuario)
    pedido.agregar_item(producto, cantidad=2)
    
    # Assert (verificar)
    self.assertEqual(pedido.total, 200)
```

### 2. Un assert por test

```python
# ❌ Malo
def test_usuario(self):
    usuario = crear_usuario()
    self.assertEqual(usuario.nombre, 'Ana')
    self.assertTrue(usuario.activo)
    self.assertIsNotNone(usuario.email)

# ✅ Bueno
def test_usuario_nombre(self):
    usuario = crear_usuario()
    self.assertEqual(usuario.nombre, 'Ana')

def test_usuario_activo(self):
    usuario = crear_usuario()
    self.assertTrue(usuario.activo)

def test_usuario_tiene_email(self):
    usuario = crear_usuario()
    self.assertIsNotNone(usuario.email)
```

### 3. Nombres descriptivos

```python
# ❌ Malo
def test_1(self):
    pass

# ✅ Bueno
def test_usuario_puede_crear_pedido_si_activo(self):
    pass

def test_usuario_no_puede_crear_pedido_si_inactivo(self):
    pass
```

### 4. Tests independientes

```python
# ❌ Malo (depende del orden)
def test_1_crear_usuario(self):
    self.usuario = crear_usuario()

def test_2_actualizar_usuario(self):
    self.usuario.nombre = 'Nuevo'  # ¡Depende del test anterior!

# ✅ Bueno
def test_crear_usuario(self):
    usuario = crear_usuario()
    self.assertIsNotNone(usuario)

def test_actualizar_usuario(self):
    usuario = crear_usuario()
    usuario.nombre = 'Nuevo'
    usuario.save()
    self.assertEqual(usuario.nombre, 'Nuevo')
```

---

## Resumen

Has aprendido:

✅ TestCase y assertions  
✅ Testing de models  
✅ Testing de views  
✅ APITestCase para DRF  
✅ Testing de autenticación  
✅ Fixtures y factories  
✅ Mocking  
✅ Coverage  
✅ pytest-django

**Siguiente:** Deployment en Django

---

## Recursos

- **[Testing in Django](https://docs.djangoproject.com/en/5.0/topics/testing/)** - Docs oficiales
- **[pytest-django](https://pytest-django.readthedocs.io/)** - Testing con pytest
- **[factory_boy](https://factoryboy.readthedocs.io/)** - Factories avanzadas

Los tests dan confianza. 🧪

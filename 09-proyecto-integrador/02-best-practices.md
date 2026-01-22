# Best Practices y Código Limpio

> **Checklist de calidad profesional para tu proyecto FullStack**

---

## Código Limpio

### Nombres descriptivos

```python
# ❌ Mal
def calc(x, y):
    return x * y * 0.19

# ✅ Bien
def calcular_impuesto(precio_base, cantidad):
    TASA_IMPUESTO = 0.19
    return precio_base * cantidad * TASA_IMPUESTO
```

```javascript
// ❌ Mal
const d = new Date();
const u = users.filter(u => u.a);

// ✅ Bien
const fechaActual = new Date();
const usuariosActivos = usuarios.filter(usuario => usuario.activo);
```

---

### Funciones pequeñas

**Una función = Una responsabilidad**

```python
# ❌ Mal - Función hace muchas cosas
def procesar_orden(orden_id):
    orden = Orden.objects.get(id=orden_id)
    # Validar stock
    for item in orden.items.all():
        if item.producto.stock < item.cantidad:
            raise ValueError('Stock insuficiente')
    # Reducir stock
    for item in orden.items.all():
        item.producto.stock -= item.cantidad
        item.producto.save()
    # Enviar email
    send_mail('Orden confirmada', f'Tu orden #{orden.id}...', 'admin@example.com', [orden.usuario.email])
    # Actualizar estado
    orden.estado = 'procesando'
    orden.save()

# ✅ Bien - Separar responsabilidades
def procesar_orden(orden_id):
    orden = obtener_orden(orden_id)
    validar_stock_orden(orden)
    reducir_stock_orden(orden)
    enviar_email_confirmacion(orden)
    actualizar_estado_orden(orden, 'procesando')

def validar_stock_orden(orden):
    for item in orden.items.all():
        if item.producto.stock < item.cantidad:
            raise ValueError(f'Stock insuficiente para {item.producto.nombre}')

def reducir_stock_orden(orden):
    for item in orden.items.all():
        item.producto.stock -= item.cantidad
        item.producto.save()
```

---

### Evitar comentarios innecesarios

```python
# ❌ Mal - Comentario obvio
# Incrementar el contador
contador += 1

# ✅ Bien - Código auto-explicativo
usuarios_activos_count += 1

# ✅ Comentario útil para explicar "por qué"
# Usar timeout de 30s porque la API externa es lenta
response = requests.get(url, timeout=30)
```

---

### Manejo de errores

```python
# ❌ Mal - Errores genéricos
try:
    producto = Producto.objects.get(id=producto_id)
except:
    pass

# ✅ Bien - Errores específicos
try:
    producto = Producto.objects.get(id=producto_id)
except Producto.DoesNotExist:
    raise Http404('Producto no encontrado')
except DatabaseError as e:
    logger.error(f'Error de base de datos: {e}')
    raise
```

```javascript
// ❌ Mal - No manejar errores
const data = await fetch('/api/productos').then(r => r.json());

// ✅ Bien - Try/catch con mensajes útiles
try {
    const response = await fetch('/api/productos');
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
} catch (error) {
    console.error('Error al cargar productos:', error);
    notificar('No se pudieron cargar los productos. Intenta más tarde.');
    return [];
}
```

---

## Arquitectura de código

### DRY (Don't Repeat Yourself)

```python
# ❌ Mal - Código duplicado
if usuario.tipo == 'premium':
    descuento = precio * 0.1
    precio_final = precio - descuento
elif usuario.tipo == 'vip':
    descuento = precio * 0.2
    precio_final = precio - descuento

# ✅ Bien - Extraer lógica común
def calcular_descuento(usuario, precio):
    DESCUENTOS = {
        'premium': 0.1,
        'vip': 0.2,
        'regular': 0
    }
    porcentaje = DESCUENTOS.get(usuario.tipo, 0)
    return precio * porcentaje

descuento = calcular_descuento(usuario, precio)
precio_final = precio - descuento
```

---

### SOLID Principles

```python
# S - Single Responsibility
class Usuario:
    """Solo gestiona datos del usuario"""
    pass

class EmailService:
    """Solo envía emails"""
    def enviar_bienvenida(self, usuario):
        pass

# O - Open/Closed (extensible sin modificar)
class DescuentoStrategy(ABC):
    @abstractmethod
    def calcular(self, precio): pass

class DescuentoPremium(DescuentoStrategy):
    def calcular(self, precio):
        return precio * 0.9

# D - Dependency Inversion (depender de abstracciones)
class ProductoService:
    def __init__(self, repo: ProductoRepository):  # Interface
        self.repo = repo
```

---

## Performance

### Backend optimizations

```python
# ❌ Mal - N+1 queries
productos = Producto.objects.all()
for producto in productos:
    print(producto.categoria.nombre)  # Query por cada producto!

# ✅ Bien - select_related (ForeignKey)
productos = Producto.objects.select_related('categoria').all()
for producto in productos:
    print(producto.categoria.nombre)  # Sin queries extras

# ❌ Mal - N+1 con ManyToMany
ordenes = Orden.objects.all()
for orden in ordenes:
    print(orden.items.count())  # Query por cada orden!

# ✅ Bien - prefetch_related (ManyToMany)
ordenes = Orden.objects.prefetch_related('items').all()
for orden in ordenes:
    print(orden.items.count())
```

### Indexar campos frecuentes

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=200, db_index=True)  # Index
    precio = models.DecimalField(max_digits=10, decimal_places=2, db_index=True)
    categoria = models.ForeignKey(Categoria, on_delete=models.CASCADE, db_index=True)
```

### Pagination

```python
# ✅ Siempre paginar listas grandes
from rest_framework.pagination import PageNumberPagination

class ProductoPagination(PageNumberPagination):
    page_size = 20
    max_page_size = 100

class ProductoViewSet(viewsets.ModelViewSet):
    pagination_class = ProductoPagination
```

---

### Frontend optimizations

```jsx
// ❌ Mal - Re-renderiza todo
function ProductosList({ productos }) {
    return (
        <div>
            {productos.map(p => <ProductoCard producto={p} />)}
        </div>
    );
}

// ✅ Bien - Memorizar componentes
import { memo } from 'react';

const ProductoCard = memo(function ProductoCard({ producto }) {
    return <div>{producto.nombre}</div>;
});

function ProductosList({ productos }) {
    return (
        <div>
            {productos.map(p => <ProductoCard key={p.id} producto={p} />)}
        </div>
    );
}
```

```javascript
// ✅ Lazy loading de rutas
import { lazy, Suspense } from 'react';

const ProductosPage = lazy(() => import('./pages/ProductosPage'));
const CheckoutPage = lazy(() => import('./pages/CheckoutPage'));

function App() {
    return (
        <Suspense fallback={<div>Cargando...</div>}>
            <Routes>
                <Route path="/productos" element={<ProductosPage />} />
                <Route path="/checkout" element={<CheckoutPage />} />
            </Routes>
        </Suspense>
    );
}
```

---

## Seguridad

### Backend

```python
# ✅ Validar siempre inputs
from rest_framework import serializers

class ProductoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Producto
        fields = '__all__'
    
    def validate_precio(self, value):
        if value <= 0:
            raise serializers.ValidationError('Precio debe ser mayor a 0')
        return value
    
    def validate_stock(self, value):
        if value < 0:
            raise serializers.ValidationError('Stock no puede ser negativo')
        return value
```

```python
# ✅ Permisos en views
from rest_framework.permissions import IsAuthenticated

class OrdenViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    
    def get_queryset(self):
        # Solo ver propias órdenes (no todas!)
        return Orden.objects.filter(usuario=self.request.user)
```

```python
# ✅ Rate limiting
from django_ratelimit.decorators import ratelimit

@ratelimit(key='ip', rate='10/m')  # 10 requests por minuto
def api_view(request):
    pass
```

---

### Frontend

```javascript
// ✅ Sanitizar inputs
import DOMPurify from 'dompurify';

function MostrarDescripcion({ descripcion }) {
    const descripcionLimpia = DOMPurify.sanitize(descripcion);
    return <div dangerouslySetInnerHTML={{ __html: descripcionLimpia }} />;
}
```

```javascript
// ✅ No exponer tokens
// ❌ Mal
localStorage.setItem('token', token);  // Vulnerable a XSS

// ✅ Mejor (httpOnly cookies)
// Backend envía cookie httpOnly
// Frontend no puede acceder con JavaScript
```

---

## Testing

### Coverage mínimo

```
Unit tests: 80%+
Integration tests: 60%+
E2E tests: Critical paths
```

### Test nombres descriptivos

```python
# ❌ Mal
def test_1():
    pass

# ✅ Bien
def test_usuario_autenticado_puede_crear_orden():
    pass

def test_usuario_anonimo_no_puede_crear_orden():
    pass

def test_orden_con_stock_insuficiente_falla():
    pass
```

---

## Git

### Commits atómicos

```bash
# ❌ Mal
git commit -m "cambios"
git commit -m "fix"

# ✅ Bien - Conventional Commits
git commit -m "feat: agregar autenticación JWT"
git commit -m "fix: corregir cálculo de total en carrito"
git commit -m "refactor: separar lógica de descuentos en service"
git commit -m "test: agregar tests para checkout"
git commit -m "docs: actualizar README con instrucciones de deployment"
```

### Branches descriptivos

```bash
# ✅ Feature branches
git checkout -b feature/auth-jwt
git checkout -b feature/carrito-compras
git checkout -b feature/pago-stripe

# ✅ Fix branches
git checkout -b fix/stock-negativo
git checkout -b fix/login-error
```

---

## Documentación

### README.md completo

```markdown
# E-commerce FullStack

## Descripción
E-commerce con React + Django + PostgreSQL

## Stack
- Frontend: React 18, Vite, Tailwind CSS
- Backend: Django 5, DRF, PostgreSQL 15
- Testing: Pytest, Vitest, Playwright
- Deployment: Docker, Railway, Vercel

## Instalación

### Backend
\`\`\`bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
\`\`\`

### Frontend
\`\`\`bash
cd frontend
npm install
npm run dev
\`\`\`

## Variables de entorno

### Backend (.env)
\`\`\`
DEBUG=True
SECRET_KEY=your-secret-key
DATABASE_URL=postgresql://user:password@localhost:5432/ecommerce
ALLOWED_HOSTS=localhost,127.0.0.1
\`\`\`

### Frontend (.env)
\`\`\`
VITE_API_URL=http://localhost:8000/api
\`\`\`

## Testing

\`\`\`bash
# Backend
cd backend
pytest --cov

# Frontend
cd frontend
npm run test

# E2E
cd frontend
npx playwright test
\`\`\`

## Deploy

\`\`\`bash
docker-compose up
\`\`\`

## API Endpoints

- POST /api/usuarios/register/ - Registrar usuario
- POST /api/usuarios/login/ - Login
- GET /api/productos/ - Listar productos
- POST /api/ordenes/ - Crear orden

## Licencia
MIT
```

---

### Docstrings en código

```python
# ✅ Documentar funciones complejas
def calcular_precio_final(producto, usuario, cupon=None):
    """
    Calcula el precio final de un producto aplicando descuentos.
    
    Args:
        producto (Producto): Instancia del producto
        usuario (Usuario): Usuario que compra
        cupon (Cupon, optional): Cupón de descuento. Defaults to None.
    
    Returns:
        Decimal: Precio final con descuentos aplicados
    
    Raises:
        ValueError: Si el producto no está disponible
    
    Example:
        >>> producto = Producto.objects.get(id=1)
        >>> usuario = Usuario.objects.get(id=1)
        >>> precio = calcular_precio_final(producto, usuario)
        >>> print(precio)
        Decimal('1350.00')
    """
    if not producto.esta_disponible():
        raise ValueError('Producto no disponible')
    
    precio = producto.precio
    
    # Descuento por tipo de usuario
    if usuario.tipo == 'premium':
        precio *= Decimal('0.9')
    
    # Descuento por cupón
    if cupon and cupon.es_valido():
        precio -= cupon.descuento
    
    return precio
```

---

## Checklist final

### Código

- [ ] Nombres descriptivos (variables, funciones, clases)
- [ ] Funciones pequeñas (< 20 líneas idealmente)
- [ ] Sin código duplicado (DRY)
- [ ] Manejo correcto de errores
- [ ] Sin código comentado (usar Git)
- [ ] Logging adecuado (errores, warnings)

### Arquitectura

- [ ] Separación de responsabilidades
- [ ] SOLID principles aplicados
- [ ] Clean Architecture (si proyecto grande)
- [ ] Código testeable

### Performance

- [ ] select_related / prefetch_related en Django
- [ ] Indexes en campos frecuentes
- [ ] Paginación en listas
- [ ] Lazy loading en React
- [ ] Memoización de componentes

### Seguridad

- [ ] DEBUG=False en producción
- [ ] SECRET_KEY desde variable de entorno
- [ ] Validación de inputs (backend y frontend)
- [ ] Permisos correctos (IsAuthenticated, IsOwner)
- [ ] Rate limiting
- [ ] HTTPS (SSL)
- [ ] CORS configurado correctamente

### Testing

- [ ] Tests unitarios (80%+ coverage)
- [ ] Tests de integración
- [ ] Tests E2E (flujos críticos)
- [ ] CI/CD ejecuta tests automáticamente

### Documentación

- [ ] README.md completo
- [ ] Instrucciones de instalación
- [ ] Variables de entorno documentadas
- [ ] API endpoints documentados
- [ ] Docstrings en funciones complejas

### Git

- [ ] Commits atómicos con mensajes descriptivos
- [ ] Branches para features/fixes
- [ ] Pull requests con code review
- [ ] .gitignore correcto

### Deploy

- [ ] Docker Compose funciona
- [ ] Variables de entorno en .env
- [ ] GitHub Actions configurado
- [ ] Deploy a producción exitoso
- [ ] Health checks funcionando
- [ ] Monitoring activo (Sentry, logs)

---

## Herramientas útiles

### Linting y formatting

**Python:**
```bash
# Black (formatting)
pip install black
black .

# Flake8 (linting)
pip install flake8
flake8 .

# isort (ordenar imports)
pip install isort
isort .
```

**JavaScript:**
```bash
# ESLint
npm install --save-dev eslint
npx eslint .

# Prettier
npm install --save-dev prettier
npx prettier --write .
```

---

### Pre-commit hooks

```.pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black
  
  - repo: https://github.com/PyCQA/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
  
  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v3.0.0
    hooks:
      - id: prettier
```

```bash
# Instalar
pip install pre-commit
pre-commit install

# Ahora se ejecuta antes de cada commit
```

---

## Resumen

**Código limpio:**
- Nombres descriptivos
- Funciones pequeñas
- Sin duplicación (DRY)
- Manejo correcto de errores

**Arquitectura:**
- SOLID principles
- Separación de responsabilidades
- Código testeable

**Performance:**
- Optimizar queries (select_related)
- Paginación
- Lazy loading

**Seguridad:**
- Validar inputs
- Permisos correctos
- HTTPS
- Rate limiting

**Testing:**
- 80%+ coverage
- Tests automáticos en CI/CD

**Próximo módulo:** Portfolio y Presentación

¡Best practices completas! ✨

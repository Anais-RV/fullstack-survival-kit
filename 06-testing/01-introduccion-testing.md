# Introducción al Testing

> **Por qué testear y tipos de tests en aplicaciones fullstack**

---

## ¿Por qué testear?

**Sin tests:**
- ❌ Bugs en producción
- ❌ Miedo a refactorizar
- ❌ Regresiones (arreglas algo, rompes otra cosa)
- ❌ Integración manual lenta

**Con tests:**
- ✅ Detectar bugs antes de producción
- ✅ Refactorizar con confianza
- ✅ Documentación viva del código
- ✅ CI/CD automatizado

---

## Pirámide de tests

```
        ╱╲
       ╱E2E╲         ← Pocos, lentos, costosos
      ╱──────╲
     ╱Integr.╲       ← Moderados
    ╱──────────╲
   ╱  Unitarios ╲    ← Muchos, rápidos, baratos
  ╱──────────────╲
```

### Tests unitarios
- **Qué:** Funciones individuales
- **Ejemplo:** `suma(2, 3) === 5`
- **Velocidad:** Muy rápidos (milisegundos)
- **Cantidad:** 70% de tus tests

### Tests de integración
- **Qué:** Múltiples componentes juntos
- **Ejemplo:** API request → Base de datos → Response
- **Velocidad:** Moderados (segundos)
- **Cantidad:** 20% de tus tests

### Tests E2E (End-to-End)
- **Qué:** Flujo completo del usuario
- **Ejemplo:** Login → Crear producto → Ver lista
- **Velocidad:** Lentos (minutos)
- **Cantidad:** 10% de tus tests

---

## Tipos de tests en Fullstack

### Frontend (React)
- **Unitarios:** Funciones, hooks personalizados
- **Componentes:** Renderizado, interacciones
- **Integración:** Formulario → API mock
- **E2E:** Usuario real navegando

### Backend (Django)
- **Unitarios:** Funciones, utilidades
- **Modelos:** Validaciones, métodos
- **Serializers:** Validaciones
- **Views/APIs:** Endpoints completos
- **Integración:** API → Base de datos

---

## Herramientas

### Frontend
- **Jest** - Test runner
- **React Testing Library** - Testear componentes React
- **Vitest** - Alternativa a Jest (más rápido con Vite)
- **Playwright** - Tests E2E

### Backend
- **pytest** - Test runner de Python
- **Django TestCase** - Tests de Django
- **Factory Boy** - Crear datos de prueba
- **pytest-django** - Integración pytest con Django

---

## Anatomía de un test

```javascript
// Patrón AAA (Arrange, Act, Assert)

test('suma dos números correctamente', () => {
    // Arrange (Preparar)
    const a = 2;
    const b = 3;
    
    // Act (Actuar)
    const resultado = suma(a, b);
    
    // Assert (Afirmar)
    expect(resultado).toBe(5);
});
```

---

## TDD (Test-Driven Development)

**Flujo:**
1. ❌ Escribir test que falla (RED)
2. ✅ Escribir código mínimo para que pase (GREEN)
3. ♻️ Refactorizar (REFACTOR)

**Ejemplo:**

```javascript
// 1. RED - Test primero
test('capitalizar primera letra', () => {
    expect(capitalize('hola')).toBe('Hola');
});
// ❌ Error: capitalize is not defined

// 2. GREEN - Implementación mínima
function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.slice(1);
}
// ✅ Test pasa

// 3. REFACTOR - Mejorar sin romper el test
function capitalize(str) {
    if (!str) return '';
    return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}
// ✅ Test sigue pasando
```

---

## Cobertura de código

**Code Coverage** = % de código ejecutado por tests

```bash
# Jest
npm test -- --coverage

# Pytest
pytest --cov=productos --cov-report=html
```

**Meta realista:**
- ✅ 80%+ cobertura
- ❌ No obsesionarse con 100%

**Priorizar:**
- ✅ Lógica de negocio crítica
- ✅ Funciones complejas
- ✅ Endpoints principales
- ❌ Getters/setters triviales
- ❌ Código generado

---

## Qué testear y qué NO testear

### ✅ Testear

**Frontend:**
- Renderizado de componentes
- Interacciones del usuario (click, submit)
- Lógica de negocio (cálculos, validaciones)
- Hooks personalizados
- Flujos críticos (login, checkout)

**Backend:**
- Endpoints API (status, datos)
- Validaciones de serializers
- Lógica de negocio en modelos/views
- Permisos y autenticación
- Casos edge (inputs vacíos, negativos)

### ❌ NO testear

- Código de terceros (React, Django)
- Getters/setters triviales
- Configuración (settings.py)
- CSS puro

---

## Mocking

**Mock** = Simular dependencias externas

**¿Por qué?**
- No depender de APIs externas
- No tocar base de datos real
- Tests más rápidos

**Ejemplo:**

```javascript
// Sin mock (mal)
test('obtener usuario de API real', async () => {
    const user = await fetch('https://api.example.com/users/1');
    // ❌ Depende de internet, API externa, lento
});

// Con mock (bien)
test('obtener usuario de API mockeada', async () => {
    fetch.mockResolvedValue({ id: 1, name: 'Juan' });
    const user = await getUser(1);
    expect(user.name).toBe('Juan');
    // ✅ Rápido, predecible, sin dependencias
});
```

---

## Tests en CI/CD

```yaml
# GitHub Actions
name: Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: npm test
```

**Beneficios:**
- ✅ Tests automáticos en cada commit
- ✅ Bloquear merge si tests fallan
- ✅ Feedback inmediato

---

## Ejemplo completo: Testing fullstack

### Backend (Django)

```python
# productos/tests.py
from django.test import TestCase
from .models import Producto

class ProductoTestCase(TestCase):
    def setUp(self):
        Producto.objects.create(nombre="Laptop", precio=1200)
    
    def test_producto_tiene_nombre(self):
        laptop = Producto.objects.get(nombre="Laptop")
        self.assertEqual(laptop.precio, 1200)
```

### Frontend (React)

```jsx
// ProductoCard.test.jsx
import { render, screen } from '@testing-library/react';
import ProductoCard from './ProductoCard';

test('renderiza nombre del producto', () => {
    const producto = { id: 1, nombre: 'Laptop', precio: 1200 };
    render(<ProductoCard producto={producto} />);
    expect(screen.getByText('Laptop')).toBeInTheDocument();
});
```

### E2E (Playwright)

```javascript
// e2e/productos.spec.js
import { test, expect } from '@playwright/test';

test('crear producto', async ({ page }) => {
    await page.goto('http://localhost:5173');
    await page.click('text=Crear Producto');
    await page.fill('input[name="nombre"]', 'Laptop');
    await page.fill('input[name="precio"]', '1200');
    await page.click('button[type="submit"]');
    await expect(page.locator('text=Laptop')).toBeVisible();
});
```

---

## Mejores prácticas

✅ **Tests independientes**
- Cada test debe poder correr solo
- No depender del orden de ejecución

✅ **Nombres descriptivos**
```javascript
// ❌ Mal
test('test1', () => { ... });

// ✅ Bien
test('usuario no puede crear producto sin nombre', () => { ... });
```

✅ **Un concepto por test**
```javascript
// ❌ Mal - Testea 3 cosas
test('producto', () => {
    expect(producto.nombre).toBe('Laptop');
    expect(producto.precio).toBe(1200);
    expect(producto.stock).toBe(10);
});

// ✅ Bien - 1 cosa por test
test('producto tiene nombre correcto', () => {
    expect(producto.nombre).toBe('Laptop');
});

test('producto tiene precio correcto', () => {
    expect(producto.precio).toBe(1200);
});
```

✅ **Tests rápidos**
- Evitar `setTimeout`
- Mockear APIs externas
- Usar base de datos en memoria

✅ **Tests mantenibles**
- Page Objects para E2E
- Helpers reutilizables
- Factories para datos de prueba

---

## Roadmap de testing

### Nivel 1: Básico
- Tests unitarios de funciones
- Tests de componentes simples
- Tests de endpoints principales

### Nivel 2: Intermedio
- Tests de integración
- Mocking de APIs
- Setup/teardown de base de datos

### Nivel 3: Avanzado
- Tests E2E con Playwright
- Coverage > 80%
- CI/CD con tests automáticos
- Mutation testing

---

## Próximos módulos

1. **Testing Frontend** - Jest + React Testing Library
2. **Testing Backend** - Pytest + Django TestCase
3. **Tests de Integración** - Frontend + Backend juntos
4. **E2E con Playwright** - Tests de usuario completo

---

## Recursos

- **[Jest](https://jestjs.io/)** - Test runner JavaScript
- **[React Testing Library](https://testing-library.com/react)** - Testear React
- **[Pytest](https://pytest.org/)** - Test runner Python
- **[Django Testing](https://docs.djangoproject.com/en/5.0/topics/testing/)** - Documentación oficial
- **[Playwright](https://playwright.dev/)** - Tests E2E

¡Empecemos a testear! 🧪

# Tests E2E con Playwright

> **Tests end-to-end simulando usuarios reales navegando la aplicación**

---

## ¿Qué son tests E2E?

**E2E (End-to-End)** = Testear toda la aplicación desde la perspectiva del usuario

```
Usuario abre navegador
  → Va a http://localhost:5173
  → Hace login
  → Crea un producto
  → Ve el producto en la lista
  → Logout
```

**Ventajas:**
- ✅ Testea flujos reales del usuario
- ✅ Detecta bugs de integración
- ✅ Valida frontend + backend + BD juntos

**Desventajas:**
- ❌ Lentos (segundos/minutos)
- ❌ Frágiles (cambios en UI los rompen)
- ❌ Costosos de mantener

---

## Instalar Playwright

```bash
npm create playwright@latest
```

**Configuración interactiva:**
```
? Where to put your end-to-end tests? → e2e
? Add a GitHub Actions workflow? → No
? Install Playwright browsers? → Yes
```

### playwright.config.js

```javascript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
    testDir: './e2e',
    fullyParallel: true,
    forbidOnly: !!process.env.CI,
    retries: process.env.CI ? 2 : 0,
    workers: process.env.CI ? 1 : undefined,
    reporter: 'html',
    use: {
        baseURL: 'http://localhost:5173',
        trace: 'on-first-retry',
    },
    
    projects: [
        {
            name: 'chromium',
            use: { ...devices['Desktop Chrome'] },
        },
        {
            name: 'firefox',
            use: { ...devices['Desktop Firefox'] },
        },
        {
            name: 'webkit',
            use: { ...devices['Desktop Safari'] },
        },
    ],
    
    webServer: {
        command: 'npm run dev',
        url: 'http://localhost:5173',
        reuseExistingServer: !process.env.CI,
    },
});
```

---

## Test básico

```javascript
// e2e/home.spec.js
import { test, expect } from '@playwright/test';

test('la página principal carga', async ({ page }) => {
    await page.goto('/');
    await expect(page).toHaveTitle(/Mi App/);
});

test('muestra el navbar', async ({ page }) => {
    await page.goto('/');
    const navbar = page.locator('nav');
    await expect(navbar).toBeVisible();
});
```

**Ejecutar:**
```bash
npx playwright test
npx playwright test --ui     # Modo interactivo
npx playwright test --debug  # Debug mode
npx playwright show-report   # Ver reporte HTML
```

---

## Flujo de autenticación

```javascript
// e2e/auth.spec.js
import { test, expect } from '@playwright/test';

test.describe('Autenticación', () => {
    test('usuario puede registrarse', async ({ page }) => {
        await page.goto('/register');
        
        await page.fill('input[name="username"]', 'nuevouser');
        await page.fill('input[name="email"]', 'nuevo@example.com');
        await page.fill('input[name="password"]', 'password123');
        await page.fill('input[name="password2"]', 'password123');
        
        await page.click('button[type="submit"]');
        
        // Debe redirigir a home después de registro
        await expect(page).toHaveURL('/');
        await expect(page.locator('text=Hola, nuevouser')).toBeVisible();
    });
    
    test('usuario puede hacer login', async ({ page }) => {
        await page.goto('/login');
        
        await page.fill('input[name="username"]', 'testuser');
        await page.fill('input[name="password"]', 'password123');
        
        await page.click('button:has-text("Entrar")');
        
        await expect(page).toHaveURL('/');
        await expect(page.locator('text=Hola, testuser')).toBeVisible();
    });
    
    test('muestra error con credenciales incorrectas', async ({ page }) => {
        await page.goto('/login');
        
        await page.fill('input[name="username"]', 'testuser');
        await page.fill('input[name="password"]', 'wrongpassword');
        
        await page.click('button:has-text("Entrar")');
        
        await expect(page.locator('text=Credenciales incorrectas')).toBeVisible();
    });
    
    test('usuario puede hacer logout', async ({ page }) => {
        // Login primero
        await page.goto('/login');
        await page.fill('input[name="username"]', 'testuser');
        await page.fill('input[name="password"]', 'password123');
        await page.click('button:has-text("Entrar")');
        
        // Logout
        await page.click('button:has-text("Cerrar Sesión")');
        
        await expect(page.locator('text=Iniciar Sesión')).toBeVisible();
    });
});
```

---

## CRUD de productos

```javascript
// e2e/productos.spec.js
import { test, expect } from '@playwright/test';

test.describe('Productos', () => {
    test.beforeEach(async ({ page }) => {
        // Login antes de cada test
        await page.goto('/login');
        await page.fill('input[name="username"]', 'testuser');
        await page.fill('input[name="password"]', 'password123');
        await page.click('button:has-text("Entrar")');
        await page.waitForURL('/');
    });
    
    test('crear producto', async ({ page }) => {
        await page.goto('/productos');
        await page.click('button:has-text("Crear Producto")');
        
        await page.fill('input[name="nombre"]', 'Laptop Test');
        await page.fill('textarea[name="descripcion"]', 'Laptop para testing');
        await page.fill('input[name="precio"]', '1500');
        await page.fill('input[name="stock"]', '10');
        await page.selectOption('select[name="categoria"]', { label: 'Electrónica' });
        
        await page.click('button[type="submit"]');
        
        // Debe redirigir a lista de productos
        await expect(page).toHaveURL('/productos');
        await expect(page.locator('text=Laptop Test')).toBeVisible();
    });
    
    test('ver detalle de producto', async ({ page }) => {
        await page.goto('/productos');
        
        // Click en primer producto
        await page.click('a:has-text("Ver"):first');
        
        // Verificar que muestra detalles
        await expect(page.locator('h1')).toBeVisible();
        await expect(page.locator('text=Precio:')).toBeVisible();
        await expect(page.locator('text=Stock:')).toBeVisible();
    });
    
    test('editar producto', async ({ page }) => {
        await page.goto('/productos');
        
        // Click en botón editar del primer producto
        await page.click('button:has-text("Editar"):first');
        
        // Cambiar nombre
        await page.fill('input[name="nombre"]', 'Laptop Editada');
        await page.fill('input[name="precio"]', '1800');
        
        await page.click('button:has-text("Actualizar")');
        
        // Verificar cambio
        await expect(page).toHaveURL('/productos');
        await expect(page.locator('text=Laptop Editada')).toBeVisible();
    });
    
    test('eliminar producto', async ({ page }) => {
        await page.goto('/productos');
        
        const productoNombre = await page.locator('tbody tr:first td:nth-child(3)').textContent();
        
        // Click en eliminar y confirmar
        page.on('dialog', dialog => dialog.accept());
        await page.click('button:has-text("Eliminar"):first');
        
        // Verificar que ya no aparece
        await expect(page.locator(`text=${productoNombre}`)).not.toBeVisible();
    });
    
    test('buscar producto', async ({ page }) => {
        await page.goto('/productos');
        
        await page.fill('input[placeholder="Buscar..."]', 'Laptop');
        await page.click('button:has-text("Buscar")');
        
        // Verificar que solo muestra resultados relevantes
        await expect(page.locator('tbody tr')).toHaveCount(1);
    });
});
```

---

## Upload de archivos

```javascript
// e2e/upload.spec.js
import { test, expect } from '@playwright/test';
import path from 'path';

test.describe('Upload de imágenes', () => {
    test.beforeEach(async ({ page }) => {
        await page.goto('/login');
        await page.fill('input[name="username"]', 'testuser');
        await page.fill('input[name="password"]', 'password123');
        await page.click('button:has-text("Entrar")');
    });
    
    test('subir imagen al crear producto', async ({ page }) => {
        await page.goto('/productos/crear');
        
        // Llenar formulario
        await page.fill('input[name="nombre"]', 'Producto con Imagen');
        await page.fill('input[name="precio"]', '100');
        await page.fill('input[name="stock"]', '5');
        
        // Subir imagen
        const filePath = path.join(__dirname, 'fixtures', 'test-image.jpg');
        await page.setInputFiles('input[type="file"]', filePath);
        
        // Verificar preview
        await expect(page.locator('img[alt="Preview"]')).toBeVisible();
        
        await page.click('button[type="submit"]');
        
        // Verificar que se creó con imagen
        await expect(page).toHaveURL('/productos');
        const productoCard = page.locator('text=Producto con Imagen').locator('..');
        await expect(productoCard.locator('img')).toBeVisible();
    });
});
```

---

## Page Object Model

**Patrón para organizar tests:**

```javascript
// e2e/pages/LoginPage.js
export class LoginPage {
    constructor(page) {
        this.page = page;
        this.usernameInput = page.locator('input[name="username"]');
        this.passwordInput = page.locator('input[name="password"]');
        this.submitButton = page.locator('button[type="submit"]');
    }
    
    async goto() {
        await this.page.goto('/login');
    }
    
    async login(username, password) {
        await this.usernameInput.fill(username);
        await this.passwordInput.fill(password);
        await this.submitButton.click();
    }
}
```

```javascript
// e2e/pages/ProductosPage.js
export class ProductosPage {
    constructor(page) {
        this.page = page;
        this.crearButton = page.locator('button:has-text("Crear Producto")');
        this.searchInput = page.locator('input[placeholder="Buscar..."]');
        this.searchButton = page.locator('button:has-text("Buscar")');
    }
    
    async goto() {
        await this.page.goto('/productos');
    }
    
    async crear(producto) {
        await this.crearButton.click();
        await this.page.fill('input[name="nombre"]', producto.nombre);
        await this.page.fill('input[name="precio"]', producto.precio.toString());
        await this.page.fill('input[name="stock"]', producto.stock.toString());
        await this.page.click('button[type="submit"]');
    }
    
    async buscar(query) {
        await this.searchInput.fill(query);
        await this.searchButton.click();
    }
}
```

**Usar Page Objects:**

```javascript
// e2e/productos.spec.js
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';
import { ProductosPage } from './pages/ProductosPage';

test.describe('Productos con Page Objects', () => {
    test('crear producto', async ({ page }) => {
        const loginPage = new LoginPage(page);
        const productosPage = new ProductosPage(page);
        
        await loginPage.goto();
        await loginPage.login('testuser', 'password123');
        
        await productosPage.goto();
        await productosPage.crear({
            nombre: 'Laptop',
            precio: 1500,
            stock: 10
        });
        
        await expect(page.locator('text=Laptop')).toBeVisible();
    });
});
```

---

## Tests con API mocking

```javascript
// e2e/productos-mocked.spec.js
import { test, expect } from '@playwright/test';

test.describe('Productos con API mockeada', () => {
    test('listar productos', async ({ page }) => {
        // Interceptar request y devolver datos mock
        await page.route('**/api/productos/', async route => {
            await route.fulfill({
                status: 200,
                body: JSON.stringify([
                    { id: 1, nombre: 'Laptop Mock', precio: 1200, stock: 10 },
                    { id: 2, nombre: 'Mouse Mock', precio: 25, stock: 50 }
                ])
            });
        });
        
        await page.goto('/productos');
        
        await expect(page.locator('text=Laptop Mock')).toBeVisible();
        await expect(page.locator('text=Mouse Mock')).toBeVisible();
    });
    
    test('error de API', async ({ page }) => {
        await page.route('**/api/productos/', async route => {
            await route.fulfill({
                status: 500,
                body: JSON.stringify({ error: 'Internal Server Error' })
            });
        });
        
        await page.goto('/productos');
        
        await expect(page.locator('text=Error')).toBeVisible();
    });
});
```

---

## Screenshots y videos

```javascript
// playwright.config.js
export default defineConfig({
    use: {
        screenshot: 'only-on-failure',
        video: 'retain-on-failure',
    },
});
```

**Screenshot manual:**
```javascript
test('página de error', async ({ page }) => {
    await page.goto('/404');
    await page.screenshot({ path: 'screenshots/error-404.png' });
});
```

---

## Tests responsive

```javascript
// e2e/responsive.spec.js
import { test, expect, devices } from '@playwright/test';

test.describe('Responsive design', () => {
    test('mobile: navbar se colapsa', async ({ browser }) => {
        const context = await browser.newContext({
            ...devices['iPhone 12'],
        });
        const page = await context.newPage();
        
        await page.goto('/');
        
        // Verificar que hay botón de menú móvil
        await expect(page.locator('button[aria-label="Menu"]')).toBeVisible();
        
        await context.close();
    });
    
    test('tablet: diseño intermedio', async ({ browser }) => {
        const context = await browser.newContext({
            ...devices['iPad'],
        });
        const page = await context.newPage();
        
        await page.goto('/productos');
        
        // Grid de 2 columnas en tablet
        const productos = page.locator('.producto-card');
        await expect(productos).toHaveCount(6);
        
        await context.close();
    });
});
```

---

## CI/CD con GitHub Actions

```yaml
# .github/workflows/e2e.yml
name: E2E Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 18
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      
      - name: Run E2E tests
        run: npx playwright test
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
```

---

## Mejores prácticas

✅ **Tests independientes**
```javascript
// ❌ Mal - Depende de test anterior
test('crear producto', async ({ page }) => { ... });
test('editar producto', async ({ page }) => {
    // Asume que el test anterior creó el producto
});

// ✅ Bien - Cada test es independiente
test('crear producto', async ({ page }) => { ... });
test('editar producto', async ({ page }) => {
    // Crea su propio producto primero
    await crearProducto();
    // Luego lo edita
});
```

✅ **Esperar elementos correctamente**
```javascript
// ❌ Mal
await page.waitForTimeout(5000);

// ✅ Bien
await page.waitForSelector('text=Producto creado');
await expect(page.locator('text=Producto creado')).toBeVisible();
```

✅ **Limpiar datos después**
```javascript
test.afterEach(async ({ page }) => {
    // Limpiar productos creados en el test
    await page.request.delete('/api/productos/test-cleanup');
});
```

✅ **Usar data-testid para elementos dinámicos**
```jsx
// Componente React
<div data-testid="producto-card-1">...</div>

// Test Playwright
await page.locator('[data-testid="producto-card-1"]').click();
```

---

## Resumen

**Playwright:**
- Tests E2E reales
- Multi-browser (Chrome, Firefox, Safari)
- Screenshots y videos
- Network mocking

**Cuándo usar E2E:**
- Flujos críticos (login, checkout)
- Integración frontend + backend
- Validar experiencia del usuario

**Cuándo NO usar E2E:**
- Tests unitarios (usar Jest/Vitest)
- Validaciones simples
- Lógica de negocio (usar pytest)

**Próximo módulo:** README de testing

---

## Recursos

- **[Playwright](https://playwright.dev/)** - Documentación oficial
- **[Playwright Best Practices](https://playwright.dev/docs/best-practices)** - Guía oficial
- **[Page Object Model](https://playwright.dev/docs/pom)** - Patrón de diseño

¡Tests E2E listos! 🎭

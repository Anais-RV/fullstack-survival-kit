# Testing Frontend (Jest + React Testing Library)

> **Testear componentes React, hooks y lógica del frontend**

---

## Configuración inicial

### Vite + Vitest

**Vitest** = Jest pero optimizado para Vite (más rápido)

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

### vite.config.js

```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [react()],
    test: {
        globals: true,
        environment: 'jsdom',
        setupFiles: './src/tests/setup.js',
    },
});
```

### setup.js

```javascript
// src/tests/setup.js
import { expect, afterEach } from 'vitest';
import { cleanup } from '@testing-library/react';
import * as matchers from '@testing-library/jest-dom/matchers';

expect.extend(matchers);

afterEach(() => {
    cleanup();
});
```

### package.json

```json
{
    "scripts": {
        "test": "vitest",
        "test:ui": "vitest --ui",
        "test:coverage": "vitest --coverage"
    }
}
```

---

## Tests de componentes

### Componente simple

```jsx
// src/components/Greeting.jsx
export default function Greeting({ name }) {
    return <h1>Hola, {name}!</h1>;
}
```

```jsx
// src/components/Greeting.test.jsx
import { render, screen } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import Greeting from './Greeting';

describe('Greeting', () => {
    it('renderiza el nombre correctamente', () => {
        render(<Greeting name="Juan" />);
        expect(screen.getByText('Hola, Juan!')).toBeInTheDocument();
    });
    
    it('renderiza sin nombre', () => {
        render(<Greeting name="" />);
        expect(screen.getByText('Hola, !')).toBeInTheDocument();
    });
});
```

### Queries de React Testing Library

```jsx
// Buscar por texto
screen.getByText('Hola');
screen.getByText(/hola/i);  // Case insensitive

// Buscar por role
screen.getByRole('button', { name: /enviar/i });
screen.getByRole('heading', { level: 1 });

// Buscar por label
screen.getByLabelText('Email:');

// Buscar por placeholder
screen.getByPlaceholderText('Buscar...');

// Buscar por test ID
screen.getByTestId('product-card');

// get vs query vs find
screen.getByText('Texto');      // Error si no existe
screen.queryByText('Texto');    // null si no existe
await screen.findByText('Texto'); // Espera a que aparezca (async)
```

---

## Tests de interacciones

### Click en botón

```jsx
// src/components/Counter.jsx
import { useState } from 'react';

export default function Counter() {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <p>Contador: {count}</p>
            <button onClick={() => setCount(count + 1)}>
                Incrementar
            </button>
        </div>
    );
}
```

```jsx
// src/components/Counter.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect } from 'vitest';
import Counter from './Counter';

describe('Counter', () => {
    it('incrementa al hacer click', async () => {
        const user = userEvent.setup();
        render(<Counter />);
        
        expect(screen.getByText('Contador: 0')).toBeInTheDocument();
        
        await user.click(screen.getByRole('button', { name: /incrementar/i }));
        
        expect(screen.getByText('Contador: 1')).toBeInTheDocument();
    });
});
```

### Formularios

```jsx
// src/components/LoginForm.jsx
import { useState } from 'react';

export default function LoginForm({ onSubmit }) {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');
    
    const handleSubmit = (e) => {
        e.preventDefault();
        onSubmit({ username, password });
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <div>
                <label htmlFor="username">Usuario:</label>
                <input
                    id="username"
                    type="text"
                    value={username}
                    onChange={(e) => setUsername(e.target.value)}
                />
            </div>
            <div>
                <label htmlFor="password">Contraseña:</label>
                <input
                    id="password"
                    type="password"
                    value={password}
                    onChange={(e) => setPassword(e.target.value)}
                />
            </div>
            <button type="submit">Entrar</button>
        </form>
    );
}
```

```jsx
// src/components/LoginForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import LoginForm from './LoginForm';

describe('LoginForm', () => {
    it('envía datos del formulario', async () => {
        const user = userEvent.setup();
        const handleSubmit = vi.fn();
        
        render(<LoginForm onSubmit={handleSubmit} />);
        
        await user.type(screen.getByLabelText(/usuario/i), 'juan');
        await user.type(screen.getByLabelText(/contraseña/i), '123456');
        await user.click(screen.getByRole('button', { name: /entrar/i }));
        
        expect(handleSubmit).toHaveBeenCalledWith({
            username: 'juan',
            password: '123456'
        });
    });
});
```

---

## Mocking

### Mockear API calls

```jsx
// src/components/ProductosList.jsx
import { useState, useEffect } from 'react';
import { productoService } from '../services/productoService';

export default function ProductosList() {
    const [productos, setProductos] = useState([]);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        const cargar = async () => {
            const result = await productoService.getAll();
            if (result.success) {
                setProductos(result.data);
            }
            setLoading(false);
        };
        cargar();
    }, []);
    
    if (loading) return <div>Cargando...</div>;
    
    return (
        <ul>
            {productos.map(p => (
                <li key={p.id}>{p.nombre}</li>
            ))}
        </ul>
    );
}
```

```jsx
// src/components/ProductosList.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import ProductosList from './ProductosList';
import { productoService } from '../services/productoService';

// Mock del servicio
vi.mock('../services/productoService');

describe('ProductosList', () => {
    beforeEach(() => {
        vi.clearAllMocks();
    });
    
    it('muestra loading inicialmente', () => {
        productoService.getAll.mockResolvedValue({ success: false, data: [] });
        render(<ProductosList />);
        expect(screen.getByText('Cargando...')).toBeInTheDocument();
    });
    
    it('renderiza lista de productos', async () => {
        const mockProductos = [
            { id: 1, nombre: 'Laptop' },
            { id: 2, nombre: 'Mouse' }
        ];
        
        productoService.getAll.mockResolvedValue({
            success: true,
            data: mockProductos
        });
        
        render(<ProductosList />);
        
        await waitFor(() => {
            expect(screen.getByText('Laptop')).toBeInTheDocument();
            expect(screen.getByText('Mouse')).toBeInTheDocument();
        });
    });
    
    it('maneja error de API', async () => {
        productoService.getAll.mockResolvedValue({ success: false, data: [] });
        
        render(<ProductosList />);
        
        await waitFor(() => {
            expect(screen.queryByText('Cargando...')).not.toBeInTheDocument();
        });
    });
});
```

---

## Tests de hooks personalizados

### Hook personalizado

```javascript
// src/hooks/useCounter.js
import { useState } from 'react';

export function useCounter(initialValue = 0) {
    const [count, setCount] = useState(initialValue);
    
    const increment = () => setCount(c => c + 1);
    const decrement = () => setCount(c => c - 1);
    const reset = () => setCount(initialValue);
    
    return { count, increment, decrement, reset };
}
```

```javascript
// src/hooks/useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { useCounter } from './useCounter';

describe('useCounter', () => {
    it('inicia con valor inicial', () => {
        const { result } = renderHook(() => useCounter(5));
        expect(result.current.count).toBe(5);
    });
    
    it('incrementa correctamente', () => {
        const { result } = renderHook(() => useCounter(0));
        
        act(() => {
            result.current.increment();
        });
        
        expect(result.current.count).toBe(1);
    });
    
    it('decrementa correctamente', () => {
        const { result } = renderHook(() => useCounter(5));
        
        act(() => {
            result.current.decrement();
        });
        
        expect(result.current.count).toBe(4);
    });
    
    it('resetea a valor inicial', () => {
        const { result } = renderHook(() => useCounter(5));
        
        act(() => {
            result.current.increment();
            result.current.increment();
        });
        
        expect(result.current.count).toBe(7);
        
        act(() => {
            result.current.reset();
        });
        
        expect(result.current.count).toBe(5);
    });
});
```

---

## Tests con Context

### Context de autenticación

```jsx
// src/context/AuthContext.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { AuthProvider, useAuth } from './AuthContext';
import { authService } from '../services/authService';

vi.mock('../services/authService');

function TestComponent() {
    const { user, login, logout } = useAuth();
    
    return (
        <div>
            {user ? (
                <>
                    <p>Usuario: {user.username}</p>
                    <button onClick={logout}>Logout</button>
                </>
            ) : (
                <button onClick={() => login('juan', '123')}>Login</button>
            )}
        </div>
    );
}

describe('AuthContext', () => {
    it('permite login', async () => {
        const user = userEvent.setup();
        
        authService.login.mockResolvedValue({
            success: true,
            data: { access: 'token123', refresh: 'refresh123' }
        });
        
        authService.getProfile.mockResolvedValue({
            id: 1,
            username: 'juan'
        });
        
        render(
            <AuthProvider>
                <TestComponent />
            </AuthProvider>
        );
        
        await user.click(screen.getByRole('button', { name: /login/i }));
        
        expect(await screen.findByText('Usuario: juan')).toBeInTheDocument();
    });
});
```

---

## Tests de componentes con Router

```jsx
// src/components/ProductoDetail.test.jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter, Route, Routes } from 'react-router-dom';
import { describe, it, expect, vi } from 'vitest';
import ProductoDetail from './ProductoDetail';
import { productoService } from '../services/productoService';

vi.mock('../services/productoService');

describe('ProductoDetail', () => {
    it('muestra detalle del producto', async () => {
        const mockProducto = {
            id: 1,
            nombre: 'Laptop',
            precio: 1200,
            descripcion: 'Laptop profesional'
        };
        
        productoService.getById.mockResolvedValue({
            success: true,
            data: mockProducto
        });
        
        render(
            <MemoryRouter initialEntries={['/productos/1']}>
                <Routes>
                    <Route path="/productos/:id" element={<ProductoDetail />} />
                </Routes>
            </MemoryRouter>
        );
        
        expect(await screen.findByText('Laptop')).toBeInTheDocument();
        expect(screen.getByText('$1200')).toBeInTheDocument();
    });
});
```

---

## Snapshot testing

```jsx
// src/components/ProductoCard.test.jsx
import { render } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import ProductoCard from './ProductoCard';

describe('ProductoCard', () => {
    it('coincide con snapshot', () => {
        const producto = {
            id: 1,
            nombre: 'Laptop',
            precio: 1200
        };
        
        const { container } = render(<ProductoCard producto={producto} />);
        expect(container).toMatchSnapshot();
    });
});
```

---

## Coverage

```bash
# Ejecutar tests con coverage
npm run test:coverage

# Ver reporte HTML
open coverage/index.html
```

**Interpretar resultados:**
- **Statements:** % de líneas ejecutadas
- **Branches:** % de if/else ejecutados
- **Functions:** % de funciones llamadas
- **Lines:** % de líneas cubiertas

**Meta:** 80%+ coverage

---

## Mejores prácticas

✅ **Testear comportamiento, no implementación**
```jsx
// ❌ Mal - Depende de estado interno
expect(component.state.count).toBe(1);

// ✅ Bien - Testea lo que ve el usuario
expect(screen.getByText('Contador: 1')).toBeInTheDocument();
```

✅ **Nombres descriptivos**
```javascript
// ❌ Mal
it('funciona', () => { ... });

// ✅ Bien
it('muestra mensaje de error cuando el email es inválido', () => { ... });
```

✅ **Arrange, Act, Assert**
```javascript
it('incrementa contador', async () => {
    // Arrange
    const user = userEvent.setup();
    render(<Counter />);
    
    // Act
    await user.click(screen.getByRole('button'));
    
    // Assert
    expect(screen.getByText('Contador: 1')).toBeInTheDocument();
});
```

✅ **Limpiar mocks**
```javascript
beforeEach(() => {
    vi.clearAllMocks();
});
```

---

## Resumen

**Setup:**
- Vitest (test runner)
- React Testing Library (testear React)
- @testing-library/user-event (simular interacciones)

**Tests de:**
- Componentes (render, props)
- Interacciones (click, type, submit)
- Formularios (inputs, validación)
- Hooks personalizados
- Context API
- Router

**Mocking:**
- API calls con `vi.mock()`
- Datos de prueba
- Funciones con `vi.fn()`

**Próximo módulo:** Testing Backend con Pytest

---

## Recursos

- **[Vitest](https://vitest.dev/)** - Test runner
- **[React Testing Library](https://testing-library.com/react)** - Documentación oficial
- **[Testing Library Cheatsheet](https://testing-library.com/docs/react-testing-library/cheatsheet)** - Queries rápidas
- **[Common Mistakes](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)** - Kent C. Dodds

¡Frontend testeado! ✅

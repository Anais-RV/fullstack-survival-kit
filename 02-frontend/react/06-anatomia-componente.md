# Módulo 06: Anatomía de un Componente React

## Estructura completa de un componente

Un componente React tiene partes bien definidas:

```jsx
// 1. IMPORTS
import { useState } from 'react';
import './Button.css';

// 2. COMPONENTE (función)
function Button({ text, onClick, disabled = false }) {
  // 3. LÓGICA (hooks, variables, funciones)
  const [clicked, setClicked] = useState(false);
  
  const handleClick = () => {
    setClicked(true);
    onClick?.();
  };
  
  // 4. RENDER (return con JSX)
  return (
    <button 
      onClick={handleClick}
      disabled={disabled}
      className={clicked ? 'clicked' : ''}
    >
      {text}
    </button>
  );
}

// 5. EXPORT
export default Button;
```

---

## 1. Imports

### Importar de React

```jsx
// Hook específico
import { useState } from 'react';

// Múltiples hooks
import { useState, useEffect, useRef } from 'react';

// React completo (raro)
import React, { useState } from 'react';
```

### Importar componentes

```jsx
// Default export
import Button from './Button';
import UserCard from './components/UserCard';

// Named exports
import { Header, Footer } from './components/Layout';

// Renombrar al importar
import { Button as PrimaryButton } from './components/Buttons';
```

### Importar estilos

```jsx
// CSS global
import './styles/global.css';

// CSS module
import styles from './Button.module.css';

// Múltiples estilos
import './Button.css';
import './Theme.css';
```

### Importar assets

```jsx
// Imágenes
import logo from './assets/logo.png';
import icon from '../images/icon.svg';

// Uso:
<img src={logo} alt="Logo" />
```

### Importar librerías

```jsx
import axios from 'axios';
import { format } from 'date-fns';
import _ from 'lodash';
```

---

## 2. Definición del componente

### Función normal (recomendado)

```jsx
function MiComponente() {
  return <div>Hola</div>;
}
```

### Arrow function

```jsx
const MiComponente = () => {
  return <div>Hola</div>;
};
```

### Arrow con return implícito

```jsx
const MiComponente = () => <div>Hola</div>;
```

### Con props

```jsx
// Props como objeto
function Saludo(props) {
  return <h1>Hola {props.nombre}</h1>;
}

// Props con destructuring (más común)
function Saludo({ nombre, edad }) {
  return (
    <div>
      <h1>Hola {nombre}</h1>
      <p>Edad: {edad}</p>
    </div>
  );
}
```

---

## 3. Lógica del componente

### Variables locales

```jsx
function Card() {
  const titulo = "Mi Card";
  const activo = true;
  const items = [1, 2, 3];
  
  return <div>{titulo}</div>;
}
```

### Funciones auxiliares

```jsx
function Calculator() {
  const sumar = (a, b) => a + b;
  const restar = (a, b) => a - b;
  
  const calcular = (operacion, x, y) => {
    if (operacion === 'suma') return sumar(x, y);
    if (operacion === 'resta') return restar(x, y);
  };
  
  return <div>{calcular('suma', 5, 3)}</div>;
}
```

### Constantes derivadas

```jsx
function UserProfile({ user }) {
  const nombreCompleto = `${user.nombre} ${user.apellido}`;
  const edad = new Date().getFullYear() - user.añoNacimiento;
  const esAdulto = edad >= 18;
  
  return (
    <div>
      <h1>{nombreCompleto}</h1>
      <p>Edad: {edad}</p>
      {esAdulto && <span>Mayor de edad</span>}
    </div>
  );
}
```

### Event handlers

```jsx
function Form() {
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Formulario enviado');
  };
  
  const handleChange = (e) => {
    console.log('Input cambió:', e.target.value);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} />
      <button type="submit">Enviar</button>
    </form>
  );
}
```

---

## 4. Return (JSX)

### Return simple

```jsx
function Simple() {
  return <div>Contenido</div>;
}
```

### Return con paréntesis (multilínea)

```jsx
function Multilínea() {
  return (
    <div>
      <h1>Título</h1>
      <p>Párrafo</p>
    </div>
  );
}
```

### Return condicional

```jsx
function Condicional({ mostrar }) {
  if (!mostrar) {
    return null;
  }
  
  return <div>Visible</div>;
}
```

### Multiple returns

```jsx
function Estado({ status }) {
  if (status === 'loading') {
    return <div>Cargando...</div>;
  }
  
  if (status === 'error') {
    return <div>Error</div>;
  }
  
  return <div>Éxito</div>;
}
```

---

## 5. Exports

### Default export (uno por archivo)

```jsx
function Button() {
  return <button>Click</button>;
}

export default Button;

// O en la misma línea:
export default function Button() {
  return <button>Click</button>;
}
```

### Named exports (múltiples)

```jsx
export function Header() {
  return <header>Header</header>;
}

export function Footer() {
  return <footer>Footer</footer>;
}

// Importar:
import { Header, Footer } from './Layout';
```

### Mixto

```jsx
function Main() {
  return <main>Main</main>;
}

export function Header() {
  return <header>Header</header>;
}

export default Main;

// Importar:
import Main, { Header } from './Layout';
```

---

## Organización de código

### ✅ Buena estructura

```jsx
import { useState, useEffect } from 'react';
import { formatDate } from '../utils/format';
import './UserCard.css';

function UserCard({ user }) {
  // 1. Hooks primero
  const [expanded, setExpanded] = useState(false);
  
  useEffect(() => {
    console.log('User cambió');
  }, [user]);
  
  // 2. Variables y constantes
  const fullName = `${user.firstName} ${user.lastName}`;
  const registeredDate = formatDate(user.createdAt);
  
  // 3. Funciones
  const toggleExpand = () => {
    setExpanded(!expanded);
  };
  
  // 4. Return
  return (
    <div className="user-card">
      <h2>{fullName}</h2>
      <p>Registrado: {registeredDate}</p>
      <button onClick={toggleExpand}>
        {expanded ? 'Ocultar' : 'Ver más'}
      </button>
      {expanded && (
        <div className="details">
          <p>Email: {user.email}</p>
          <p>Teléfono: {user.phone}</p>
        </div>
      )}
    </div>
  );
}

export default UserCard;
```

---

## Componentes como archivos

### Un componente = un archivo

```
components/
├── Button.jsx
├── Button.css
├── Card.jsx
├── Card.css
└── Header.jsx
```

### Carpeta por componente (componentes complejos)

```
components/
└── UserProfile/
    ├── index.jsx          ← Componente principal
    ├── UserProfile.css
    ├── Avatar.jsx         ← Subcomponentes
    └── Stats.jsx
```

**Importar:**
```jsx
// Importa automáticamente index.jsx
import UserProfile from './components/UserProfile';
```

---

## Tipos de componentes

### 1. Presentacional (solo UI)

```jsx
function Avatar({ src, alt, size = 'medium' }) {
  return (
    <img 
      src={src} 
      alt={alt}
      className={`avatar avatar-${size}`}
    />
  );
}
```

### 2. Contenedor (con lógica)

```jsx
import { useState, useEffect } from 'react';
import UserCard from './UserCard';

function UserContainer({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Cargando...</div>;
  
  return <UserCard user={user} />;
}
```

### 3. Layout

```jsx
function PageLayout({ children }) {
  return (
    <div className="page">
      <header>
        <h1>Mi App</h1>
      </header>
      <main>{children}</main>
      <footer>© 2026</footer>
    </div>
  );
}
```

---

## Convenciones de nombres

### Componentes: PascalCase

```jsx
// ✅ Correcto
function UserProfile() {}
function ProductCard() {}
function NavBar() {}

// ❌ Incorrecto
function userProfile() {}
function product_card() {}
function navbar() {}
```

### Archivos: PascalCase o kebab-case

```
✅ UserProfile.jsx
✅ user-profile.jsx
❌ userProfile.jsx
❌ user_profile.jsx
```

### Props y variables: camelCase

```jsx
function Card({ userName, isActive, imageUrl }) {
  const cardTitle = "Título";
  const hasContent = true;
  
  return <div>{userName}</div>;
}
```

### Event handlers: handle + Acción

```jsx
function Form() {
  const handleSubmit = () => {};
  const handleChange = () => {};
  const handleClick = () => {};
  const handleFocus = () => {};
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

### Boolean props: is/has/can

```jsx
function Button({ isDisabled, hasIcon, canSubmit }) {
  return <button disabled={isDisabled}>Click</button>;
}

// Uso:
<Button isDisabled hasIcon canSubmit />
```

---

## Comentarios en componentes

```jsx
function Component() {
  // Comentario de una línea
  
  /*
   * Comentario
   * multilínea
   */
  
  return (
    <div>
      {/* Comentario dentro de JSX */}
      <p>Texto</p>
      
      {/* 
        Comentario JSX
        multilínea
      */}
    </div>
  );
}
```

---

## Props por defecto

### Con destructuring

```jsx
function Button({ text = 'Click', size = 'medium', disabled = false }) {
  return (
    <button className={`btn btn-${size}`} disabled={disabled}>
      {text}
    </button>
  );
}
```

### Con defaultProps (método antiguo)

```jsx
function Button({ text, size }) {
  return <button className={`btn-${size}`}>{text}</button>;
}

Button.defaultProps = {
  text: 'Click',
  size: 'medium'
};
```

---

## Validación de props (opcional)

Con PropTypes:

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, email }) {
  return <div>{name}</div>;
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  email: PropTypes.string.isRequired
};
```

**Nota:** PropTypes se usan poco ahora. TypeScript es mejor opción.

---

## Ejemplo completo: ProductCard

```jsx
// ProductCard.jsx
import { useState } from 'react';
import './ProductCard.css';

/**
 * Tarjeta de producto con imagen, título, precio y botón
 * @param {Object} props - Props del componente
 * @param {string} props.name - Nombre del producto
 * @param {number} props.price - Precio en dólares
 * @param {string} props.image - URL de la imagen
 * @param {boolean} props.inStock - Si está disponible
 */
function ProductCard({ name, price, image, inStock = true }) {
  // Estado
  const [quantity, setQuantity] = useState(1);
  const [added, setAdded] = useState(false);
  
  // Constantes derivadas
  const total = price * quantity;
  const formattedPrice = `$${price.toFixed(2)}`;
  const formattedTotal = `$${total.toFixed(2)}`;
  
  // Handlers
  const handleIncrement = () => {
    if (quantity < 10) setQuantity(quantity + 1);
  };
  
  const handleDecrement = () => {
    if (quantity > 1) setQuantity(quantity - 1);
  };
  
  const handleAddToCart = () => {
    console.log(`Agregado: ${name} x${quantity}`);
    setAdded(true);
    setTimeout(() => setAdded(false), 2000);
  };
  
  // Render
  return (
    <div className={`product-card ${!inStock ? 'out-of-stock' : ''}`}>
      <img src={image} alt={name} className="product-image" />
      
      <div className="product-info">
        <h3 className="product-name">{name}</h3>
        <p className="product-price">{formattedPrice}</p>
        
        {!inStock && <span className="badge">Agotado</span>}
        
        {inStock && (
          <>
            <div className="quantity-control">
              <button onClick={handleDecrement}>-</button>
              <span>{quantity}</span>
              <button onClick={handleIncrement}>+</button>
            </div>
            
            <p className="total">Total: {formattedTotal}</p>
            
            <button 
              onClick={handleAddToCart}
              className={`add-btn ${added ? 'added' : ''}`}
              disabled={added}
            >
              {added ? '✓ Agregado' : 'Agregar al carrito'}
            </button>
          </>
        )}
      </div>
    </div>
  );
}

export default ProductCard;
```

---

## Buenas prácticas

✅ **Un componente por archivo**  
✅ **Imports ordenados**: React → Librerías → Componentes → Estilos  
✅ **Hooks al inicio**  
✅ **Funciones antes del return**  
✅ **Nombres descriptivos**  
✅ **Componentes pequeños (< 250 líneas)**  
✅ **Extraer lógica compleja a custom hooks**  
✅ **Props por defecto con destructuring**  

---

## Errores comunes

### 1. Export/Import desalineados

```jsx
// Button.jsx
export function Button() {}  // Named export

// App.jsx
import Button from './Button';  // ❌ Busca default export

// ✅ Correcto:
import { Button } from './Button';
```

### 2. Hooks después de condicionales

```jsx
// ❌ Incorrecto
function Component({ show }) {
  if (show) {
    const [value, setValue] = useState(0);  // Error
  }
}

// ✅ Correcto
function Component({ show }) {
  const [value, setValue] = useState(0);
  
  if (!show) return null;
}
```

### 3. Return sin paréntesis multilínea

```jsx
// ❌ Error - return solo toma <div>
return 
  <div>
    <h1>Título</h1>
  </div>

// ✅ Correcto
return (
  <div>
    <h1>Título</h1>
  </div>
);
```

---

## Resumen

✅ **Componente = Imports + Función + Lógica + Return + Export**  
✅ **Organización: Hooks → Variables → Funciones → Return**  
✅ **PascalCase para componentes**  
✅ **camelCase para props y variables**  
✅ **Default export para componente principal**  
✅ **Named exports para utilidades**

---

## Siguiente paso

Entiendes la anatomía completa. Ahora profundizarás en **props**.

**Siguiente:** [Módulo 07 - Props](./07-props.md)

**Anterior:** [Módulo 05 - JSX en profundidad](./05-jsx-profundidad.md)

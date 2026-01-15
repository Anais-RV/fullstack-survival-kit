# Módulo 07: Props en React

## ¿Qué son las props?

**Props** (propiedades) son datos que pasas de un componente **padre** a un componente **hijo**.

### Analogía: Argumentos de función

```jsx
// Función JavaScript
function saludar(nombre, edad) {
  return `Hola ${nombre}, tienes ${edad} años`;
}
saludar("Ana", 25);

// Componente React
function Saludo({ nombre, edad }) {
  return <p>Hola {nombre}, tienes {edad} años</p>;
}
<Saludo nombre="Ana" edad={25} />
```

---

## Pasar props

```jsx
function App() {
  return (
    <div>
      <Saludo nombre="Ana" edad={25} />
      <Saludo nombre="Luis" edad={30} />
    </div>
  );
}

function Saludo({ nombre, edad }) {
  return (
    <div>
      <h2>Hola, {nombre}</h2>
      <p>Tienes {edad} años</p>
    </div>
  );
}
```

---

## Recibir props

### 1. Como objeto

```jsx
function Saludo(props) {
  return <h1>Hola, {props.nombre}</h1>;
}

// Uso:
<Saludo nombre="Ana" />
```

### 2. Con destructuring (recomendado)

```jsx
function Saludo({ nombre, edad }) {
  return (
    <div>
      <h1>Hola, {nombre}</h1>
      <p>Edad: {edad}</p>
    </div>
  );
}
```

### 3. Con destructuring y resto

```jsx
function Button({ text, ...props }) {
  return <button {...props}>{text}</button>;
}

// Uso:
<Button text="Click" className="primary" disabled />
```

---

## Tipos de props

### String (sin llaves)

```jsx
<Saludo nombre="Ana" apellido="García" />
```

### Number (con llaves)

```jsx
<ProductCard precio={99.99} stock={5} />
```

### Boolean

```jsx
// Implícito (true)
<Button disabled primary />

// Explícito
<Button disabled={true} primary={false} />
```

### Array

```jsx
<List items={[1, 2, 3, 4, 5]} />
<Tags tags={['react', 'javascript', 'web']} />
```

### Object

```jsx
<UserCard user={{ nombre: "Ana", edad: 25, email: "ana@email.com" }} />
```

### Function

```jsx
<Button onClick={() => alert('Clicked')} />
<Form onSubmit={handleSubmit} />
```

### JSX/Component

```jsx
<Card header={<h1>Título</h1>} />
<Modal content={<UserProfile />} />
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

// Uso sin especificar:
<Button />  // text="Click", size="medium", disabled=false

// Uso personalizado:
<Button text="Enviar" size="large" />
```

### Con operador ||

```jsx
function Saludo({ nombre }) {
  const nombreFinal = nombre || 'Invitado';
  return <h1>Hola, {nombreFinal}</h1>;
}
```

### Con ??

 (nullish coalescing)

```jsx
function Contador({ valor }) {
  // || fallaría si valor es 0
  const valorFinal = valor ?? 10;  // Solo null/undefined
  return <p>Valor: {valorFinal}</p>;
}
```

---

## Children prop

`children` es una prop especial que contiene lo que pones **entre las etiquetas**:

```jsx
function Card({ children }) {
  return (
    <div className="card">
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// Uso:
<Card>
  <h2>Título</h2>
  <p>Este es el contenido</p>
</Card>
```

### Múltiples children

```jsx
function Layout({ header, sidebar, children }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}

// Uso:
<Layout 
  header={<h1>Mi App</h1>}
  sidebar={<Nav />}
>
  <p>Contenido principal</p>
</Layout>
```

---

## Props inmutables

⚠️ **Las props son de SOLO LECTURA:**

```jsx
// ❌ NUNCA hagas esto
function Component({ value }) {
  value = value + 1;  // Error: No puedes modificar props
  return <div>{value}</div>;
}

// ✅ Usa estado si necesitas modificar
function Component({ initialValue }) {
  const [value, setValue] = useState(initialValue);
  
  const increment = () => setValue(value + 1);
  
  return <div>{value}</div>;
}
```

---

## Pasar props dinámicamente

### Variables

```jsx
function App() {
  const usuario = {
    nombre: "Ana",
    edad: 25,
    rol: "Admin"
  };
  
  return <UserCard nombre={usuario.nombre} edad={usuario.edad} rol={usuario.rol} />;
}
```

### Spread operator

```jsx
function App() {
  const usuario = {
    nombre: "Ana",
    edad: 25,
    rol: "Admin"
  };
  
  // Pasa todas las propiedades como props
  return <UserCard {...usuario} />;
}

// Equivalente a:
<UserCard nombre="Ana" edad={25} rol="Admin" />
```

### Con condicionales

```jsx
function App() {
  const esAdmin = true;
  
  return (
    <Button 
      text={esAdmin ? "Panel Admin" : "Panel Usuario"}
      variant={esAdmin ? "primary" : "secondary"}
      icon={esAdmin ? "⚙️" : "👤"}
    />
  );
}
```

---

## Desestructurar props anidadas

```jsx
function UserCard({ user: { nombre, email, edad } }) {
  return (
    <div>
      <h2>{nombre}</h2>
      <p>{email}</p>
      <p>{edad} años</p>
    </div>
  );
}

// Uso:
<UserCard user={{ nombre: "Ana", email: "ana@email.com", edad: 25 }} />
```

---

## Props con funciones (callbacks)

```jsx
function Parent() {
  const handleClick = (mensaje) => {
    alert(mensaje);
  };
  
  return <Child onAction={handleClick} />;
}

function Child({ onAction }) {
  return (
    <button onClick={() => onAction('¡Hola desde Child!')}>
      Click
    </button>
  );
}
```

### Pasar datos del hijo al padre

```jsx
function Parent() {
  const handleUserSelect = (usuario) => {
    console.log('Usuario seleccionado:', usuario);
  };
  
  return <UserList onSelect={handleUserSelect} />;
}

function UserList({ onSelect }) {
  const usuarios = [
    { id: 1, nombre: "Ana" },
    { id: 2, nombre: "Luis" }
  ];
  
  return (
    <ul>
      {usuarios.map(user => (
        <li key={user.id} onClick={() => onSelect(user)}>
          {user.nombre}
        </li>
      ))}
    </ul>
  );
}
```

---

## Validar props (opcional)

Con PropTypes:

```jsx
import PropTypes from 'prop-types';

function UserCard({ nombre, edad, email, activo }) {
  return <div>{nombre}</div>;
}

UserCard.propTypes = {
  nombre: PropTypes.string.isRequired,
  edad: PropTypes.number.isRequired,
  email: PropTypes.string,
  activo: PropTypes.bool
};

UserCard.defaultProps = {
  activo: false
};
```

**Nota:** PropTypes solo valida en desarrollo. En producción, usa TypeScript.

---

## Ejemplo práctico: Sistema de tarjetas

```jsx
// App.jsx
function App() {
  const productos = [
    { id: 1, nombre: "Laptop", precio: 1200, imagen: "/laptop.jpg", stock: 5 },
    { id: 2, nombre: "Mouse", precio: 25, imagen: "/mouse.jpg", stock: 0 },
    { id: 3, nombre: "Teclado", precio: 80, imagen: "/teclado.jpg", stock: 12 }
  ];
  
  const handleAddToCart = (producto) => {
    console.log(`Agregado al carrito: ${producto.nombre}`);
  };
  
  return (
    <div className="shop">
      <h1>Tienda Online</h1>
      <div className="products">
        {productos.map(producto => (
          <ProductCard
            key={producto.id}
            {...producto}
            onAddToCart={handleAddToCart}
          />
        ))}
      </div>
    </div>
  );
}

// ProductCard.jsx
function ProductCard({ nombre, precio, imagen, stock, onAddToCart }) {
  const enStock = stock > 0;
  const precioFormateado = `$${precio.toFixed(2)}`;
  
  const handleClick = () => {
    onAddToCart({ nombre, precio });
  };
  
  return (
    <div className={`card ${!enStock ? 'agotado' : ''}`}>
      <img src={imagen} alt={nombre} />
      <h3>{nombre}</h3>
      <p className="precio">{precioFormateado}</p>
      
      {enStock ? (
        <>
          <p className="stock">Stock: {stock}</p>
          <button onClick={handleClick}>Agregar al carrito</button>
        </>
      ) : (
        <p className="agotado-badge">Agotado</p>
      )}
    </div>
  );
}
```

---

## Props vs Estado

| Props | Estado |
|-------|--------|
| Vienen del padre | Interno del componente |
| Solo lectura | Se puede modificar |
| No cambian (desde la perspectiva del hijo) | Cambia con `setState` |
| Pasan datos hacia abajo | Local al componente |

```jsx
// Props - datos del padre
function Child({ mensaje }) {
  return <p>{mensaje}</p>;
}

// Estado - datos internos
function Parent() {
  const [contador, setContador] = useState(0);
  
  return (
    <div>
      <p>Contador: {contador}</p>
      <button onClick={() => setContador(contador + 1)}>+</button>
      <Child mensaje={`El contador es ${contador}`} />
    </div>
  );
}
```

---

## Composición con props

### Botones reutilizables

```jsx
function Button({ variant = 'primary', size = 'medium', children, ...props }) {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`}
      {...props}
    >
      {children}
    </button>
  );
}

// Uso:
<Button variant="primary" size="large" onClick={handleClick}>
  Guardar
</Button>

<Button variant="secondary" disabled>
  Cancelar
</Button>
```

### Layout flexible

```jsx
function Container({ maxWidth = '1200px', padding = '20px', children }) {
  return (
    <div style={{ maxWidth, padding, margin: '0 auto' }}>
      {children}
    </div>
  );
}

function Section({ title, children }) {
  return (
    <section>
      <h2>{title}</h2>
      <div>{children}</div>
    </section>
  );
}

// Uso:
<Container maxWidth="800px">
  <Section title="Productos">
    <ProductList />
  </Section>
  <Section title="Categorías">
    <CategoryList />
  </Section>
</Container>
```

---

## Patterns comunes

### Render prop

```jsx
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);
  
  if (loading) return <p>Cargando...</p>;
  
  return render(data);
}

// Uso:
<DataFetcher
  url="/api/users"
  render={(users) => (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  )}
/>
```

### Compound components

```jsx
function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);
  
  return (
    <div className="tabs">
      {React.Children.map(children, (child, index) =>
        React.cloneElement(child, {
          isActive: index === activeTab,
          onClick: () => setActiveTab(index)
        })
      )}
    </div>
  );
}

function Tab({ title, isActive, onClick, children }) {
  return (
    <div>
      <button onClick={onClick} className={isActive ? 'active' : ''}>
        {title}
      </button>
      {isActive && <div>{children}</div>}
    </div>
  );
}

// Uso:
<Tabs>
  <Tab title="Tab 1">Contenido 1</Tab>
  <Tab title="Tab 2">Contenido 2</Tab>
  <Tab title="Tab 3">Contenido 3</Tab>
</Tabs>
```

---

## Errores comunes

### 1. Modificar props

```jsx
// ❌ Error
function Component({ items }) {
  items.push('nuevo');  // Mutando props
  return <div>{items.length}</div>;
}

// ✅ Correcto
function Component({ items }) {
  const newItems = [...items, 'nuevo'];
  return <div>{newItems.length}</div>;
}
```

### 2. Props con mismo nombre que variables

```jsx
// ❌ Confuso
function Component({ nombre }) {
  const nombre = nombre.toUpperCase();  // Error
  return <div>{nombre}</div>;
}

// ✅ Correcto
function Component({ nombre }) {
  const nombreMayusculas = nombre.toUpperCase();
  return <div>{nombreMayusculas}</div>;
}
```

### 3. Olvidar llaves en props no-string

```jsx
// ❌ Error
<Component edad="25" activo="true" />  // Son strings

// ✅ Correcto
<Component edad={25} activo={true} />
```

---

## Ejercicio: Galería de imágenes

```jsx
function App() {
  const imagenes = [
    { id: 1, url: '/img1.jpg', titulo: 'Atardecer', likes: 45 },
    { id: 2, url: '/img2.jpg', titulo: 'Montaña', likes: 32 },
    { id: 3, url: '/img3.jpg', titulo: 'Playa', likes: 78 }
  ];
  
  const handleLike = (id) => {
    console.log(`Like en imagen ${id}`);
  };
  
  return (
    <Gallery images={imagenes} onLike={handleLike} />
  );
}

function Gallery({ images, onLike }) {
  return (
    <div className="gallery">
      {images.map(img => (
        <ImageCard key={img.id} {...img} onLike={onLike} />
      ))}
    </div>
  );
}

function ImageCard({ id, url, titulo, likes, onLike }) {
  return (
    <div className="image-card">
      <img src={url} alt={titulo} />
      <h3>{titulo}</h3>
      <div className="actions">
        <span>❤️ {likes}</span>
        <button onClick={() => onLike(id)}>Me gusta</button>
      </div>
    </div>
  );
}
```

---

## Resumen

✅ **Props = datos de padre a hijo**  
✅ **Solo lectura (inmutables)**  
✅ **Destructuring recomendado**  
✅ **children es prop especial**  
✅ **Spread operator para múltiples props**  
✅ **Callbacks para comunicación hijo → padre**  
✅ **Valores por defecto con `=`**

---

## Siguiente paso

Dominas props. Ahora aprenderás **composición de componentes**.

**Siguiente:** [Módulo 08 - Composición de componentes](./08-composicion-componentes.md)

**Anterior:** [Módulo 06 - Anatomía de un componente](./06-anatomia-componente.md)

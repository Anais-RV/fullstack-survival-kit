# Módulo 08: Composición de Componentes

## ¿Qué es la composición?

**Composición** es combinar componentes pequeños para crear componentes más complejos.

### Analogía: Bloques de LEGO

```
Casa (componente grande)
├── Techo
├── Ventanas (x4)
├── Puerta
└── Cimientos

Cada parte es un bloque independiente reutilizable.
```

---

## Composición básica

```jsx
function App() {
  return (
    <div>
      <Header />
      <Main />
      <Footer />
    </div>
  );
}

function Header() {
  return (
    <header>
      <Logo />
      <Nav />
    </header>
  );
}

function Logo() {
  return <h1>Mi Sitio</h1>;
}

function Nav() {
  return (
    <nav>
      <a href="/">Inicio</a>
      <a href="/about">Acerca</a>
    </nav>
  );
}

function Main() {
  return <main>Contenido principal</main>;
}

function Footer() {
  return <footer>© 2026</footer>;
}
```

---

## Composición con children

### Container reutilizable

```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

// Uso:
<Card>
  <h2>Título</h2>
  <p>Contenido</p>
</Card>

<Card>
  <img src="/foto.jpg" alt="Foto" />
  <button>Ver más</button>
</Card>
```

### Layout con slots

```jsx
function PageLayout({ header, sidebar, footer, children }) {
  return (
    <div className="layout">
      <header className="header">{header}</header>
      <aside className="sidebar">{sidebar}</aside>
      <main className="content">{children}</main>
      <footer className="footer">{footer}</footer>
    </div>
  );
}

// Uso:
<PageLayout
  header={<Header />}
  sidebar={<Sidebar />}
  footer={<Footer />}
>
  <h1>Página Principal</h1>
  <p>Contenido...</p>
</PageLayout>
```

---

## Composición vs Herencia

React favorece **composición** sobre herencia.

### ❌ Herencia (no recomendado)

```jsx
// No hagas esto en React
class BaseCard extends Component {
  render() {
    return <div className="card">{this.props.children}</div>;
  }
}

class UserCard extends BaseCard {
  render() {
    return (
      <div>
        {super.render()}
        <p>Usuario: {this.props.name}</p>
      </div>
    );
  }
}
```

### ✅ Composición (recomendado)

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function UserCard({ name }) {
  return (
    <Card>
      <p>Usuario: {name}</p>
    </Card>
  );
}
```

---

## Patrones de composición

### 1. Container + Presentational

**Container** (lógica):
```jsx
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
  
  if (loading) return <Loading />;
  
  return <UserView user={user} />;
}
```

**Presentational** (UI):
```jsx
function UserView({ user }) {
  return (
    <div className="user">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### 2. Compound Components

Componentes que trabajan juntos:

```jsx
function Accordion({ children }) {
  const [openIndex, setOpenIndex] = useState(null);
  
  return (
    <div className="accordion">
      {React.Children.map(children, (child, index) =>
        React.cloneElement(child, {
          isOpen: openIndex === index,
          onToggle: () => setOpenIndex(openIndex === index ? null : index)
        })
      )}
    </div>
  );
}

function AccordionItem({ title, isOpen, onToggle, children }) {
  return (
    <div className="accordion-item">
      <button onClick={onToggle}>
        {title} {isOpen ? '▼' : '▶'}
      </button>
      {isOpen && <div className="content">{children}</div>}
    </div>
  );
}

// Uso:
<Accordion>
  <AccordionItem title="Sección 1">
    Contenido 1
  </AccordionItem>
  <AccordionItem title="Sección 2">
    Contenido 2
  </AccordionItem>
</Accordion>
```

### 3. Higher-Order Components (HOC)

Función que toma un componente y devuelve uno nuevo:

```jsx
function withLoading(Component) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) return <div>Cargando...</div>;
    return <Component {...props} />;
  };
}

// Uso:
const UserListWithLoading = withLoading(UserList);

<UserListWithLoading isLoading={loading} users={users} />
```

---

## Componentes especializados

### Base genérica

```jsx
function Button({ variant = 'primary', children, ...props }) {
  return (
    <button className={`btn btn-${variant}`} {...props}>
      {children}
    </button>
  );
}
```

### Especializaciones

```jsx
function PrimaryButton({ children, ...props }) {
  return <Button variant="primary" {...props}>{children}</Button>;
}

function SecondaryButton({ children, ...props }) {
  return <Button variant="secondary" {...props}>{children}</Button>;
}

function DangerButton({ children, ...props }) {
  return (
    <Button 
      variant="danger" 
      onClick={(e) => {
        if (!confirm('¿Estás seguro?')) {
          e.preventDefault();
        } else {
          props.onClick?.(e);
        }
      }}
      {...props}
    >
      {children}
    </Button>
  );
}

// Uso:
<PrimaryButton onClick={handleSave}>Guardar</PrimaryButton>
<SecondaryButton onClick={handleCancel}>Cancelar</SecondaryButton>
<DangerButton onClick={handleDelete}>Eliminar</DangerButton>
```

---

## Modal reutilizable

```jsx
function Modal({ isOpen, onClose, title, children, footer }) {
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={(e) => e.stopPropagation()}>
        <div className="modal-header">
          <h2>{title}</h2>
          <button onClick={onClose}>✕</button>
        </div>
        <div className="modal-body">
          {children}
        </div>
        {footer && (
          <div className="modal-footer">
            {footer}
          </div>
        )}
      </div>
    </div>
  );
}

// Uso:
function App() {
  const [showModal, setShowModal] = useState(false);
  
  return (
    <>
      <button onClick={() => setShowModal(true)}>Abrir Modal</button>
      
      <Modal
        isOpen={showModal}
        onClose={() => setShowModal(false)}
        title="Confirmar acción"
        footer={
          <>
            <button onClick={() => setShowModal(false)}>Cancelar</button>
            <button onClick={handleConfirm}>Confirmar</button>
          </>
        }
      >
        <p>¿Estás seguro de que quieres continuar?</p>
      </Modal>
    </>
  );
}
```

---

## Tabs component

```jsx
function Tabs({ defaultTab = 0, children }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  const tabs = React.Children.toArray(children);
  
  return (
    <div className="tabs">
      <div className="tab-headers">
        {tabs.map((tab, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            className={activeTab === index ? 'active' : ''}
          >
            {tab.props.label}
          </button>
        ))}
      </div>
      <div className="tab-content">
        {tabs[activeTab]}
      </div>
    </div>
  );
}

function TabPanel({ label, children }) {
  return <div>{children}</div>;
}

// Uso:
<Tabs defaultTab={0}>
  <TabPanel label="Perfil">
    <UserProfile />
  </TabPanel>
  <TabPanel label="Configuración">
    <Settings />
  </TabPanel>
  <TabPanel label="Notificaciones">
    <Notifications />
  </TabPanel>
</Tabs>
```

---

## Form components

```jsx
function Form({ onSubmit, children }) {
  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const data = Object.fromEntries(formData);
    onSubmit(data);
  };
  
  return <form onSubmit={handleSubmit}>{children}</form>;
}

function FormGroup({ label, children }) {
  return (
    <div className="form-group">
      <label>{label}</label>
      {children}
    </div>
  );
}

function Input({ name, type = 'text', ...props }) {
  return <input name={name} type={type} {...props} />;
}

// Uso:
<Form onSubmit={handleSubmit}>
  <FormGroup label="Nombre">
    <Input name="nombre" required />
  </FormGroup>
  <FormGroup label="Email">
    <Input name="email" type="email" required />
  </FormGroup>
  <button type="submit">Enviar</button>
</Form>
```

---

## Lista con componentes

```jsx
function List({ items, renderItem }) {
  return (
    <ul className="list">
      {items.map((item, index) => (
        <li key={item.id || index}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// Uso:
const usuarios = [
  { id: 1, nombre: "Ana", rol: "Admin" },
  { id: 2, nombre: "Luis", rol: "Usuario" }
];

<List
  items={usuarios}
  renderItem={(user) => (
    <div>
      <strong>{user.nombre}</strong> - {user.rol}
    </div>
  )}
/>
```

---

## Dashboard composición completa

```jsx
function Dashboard() {
  return (
    <PageLayout>
      <DashboardHeader />
      <DashboardContent />
    </PageLayout>
  );
}

function PageLayout({ children }) {
  return (
    <div className="page-layout">
      <Sidebar />
      <main className="main-content">
        {children}
      </main>
    </div>
  );
}

function Sidebar() {
  return (
    <aside className="sidebar">
      <Logo />
      <Nav />
    </aside>
  );
}

function DashboardHeader() {
  return (
    <header className="dashboard-header">
      <h1>Dashboard</h1>
      <UserMenu />
    </header>
  );
}

function DashboardContent() {
  return (
    <div className="dashboard-content">
      <StatsGrid />
      <Charts />
      <RecentActivity />
    </div>
  );
}

function StatsGrid() {
  const stats = [
    { id: 1, label: "Ventas", value: "$12,500", icon: "💰" },
    { id: 2, label: "Usuarios", value: "245", icon: "👥" },
    { id: 3, label: "Productos", value: "89", icon: "📦" }
  ];
  
  return (
    <div className="stats-grid">
      {stats.map(stat => (
        <StatCard key={stat.id} {...stat} />
      ))}
    </div>
  );
}

function StatCard({ label, value, icon }) {
  return (
    <Card>
      <div className="stat-card">
        <span className="icon">{icon}</span>
        <div>
          <p className="label">{label}</p>
          <p className="value">{value}</p>
        </div>
      </div>
    </Card>
  );
}

function Card({ children, className = '' }) {
  return <div className={`card ${className}`}>{children}</div>;
}
```

---

## Slot pattern

```jsx
function Dialog({ header, body, footer }) {
  return (
    <div className="dialog">
      {header && <div className="dialog-header">{header}</div>}
      {body && <div className="dialog-body">{body}</div>}
      {footer && <div className="dialog-footer">{footer}</div>}
    </div>
  );
}

// Uso:
<Dialog
  header={<h2>Confirmar eliminación</h2>}
  body={<p>¿Estás seguro de que quieres eliminar este elemento?</p>}
  footer={
    <>
      <Button variant="secondary" onClick={onCancel}>Cancelar</Button>
      <Button variant="danger" onClick={onConfirm}>Eliminar</Button>
    </>
  }
/>
```

---

## Composition con Context

```jsx
const TableContext = createContext();

function Table({ columns, data, children }) {
  return (
    <TableContext.Provider value={{ columns, data }}>
      <table className="table">
        {children}
      </table>
    </TableContext.Provider>
  );
}

function TableHeader() {
  const { columns } = useContext(TableContext);
  
  return (
    <thead>
      <tr>
        {columns.map(col => (
          <th key={col.key}>{col.label}</th>
        ))}
      </tr>
    </thead>
  );
}

function TableBody() {
  const { columns, data } = useContext(TableContext);
  
  return (
    <tbody>
      {data.map((row, i) => (
        <tr key={i}>
          {columns.map(col => (
            <td key={col.key}>{row[col.key]}</td>
          ))}
        </tr>
      ))}
    </tbody>
  );
}

// Uso:
<Table
  columns={[
    { key: 'nombre', label: 'Nombre' },
    { key: 'email', label: 'Email' }
  ]}
  data={users}
>
  <TableHeader />
  <TableBody />
</Table>
```

---

## Buenas prácticas

✅ **Componentes pequeños y enfocados**
```jsx
// ✅ Cada uno hace una cosa
<UserCard>
  <Avatar />
  <UserInfo />
  <UserActions />
</UserCard>
```

✅ **Extraer lógica común**
```jsx
// ✅ Card reutilizable
function Card({ title, children }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div>{children}</div>
    </div>
  );
}
```

✅ **Props explícitas mejor que implícitas**
```jsx
// ✅ Claro qué hace cada prop
<Modal 
  isOpen={showModal}
  onClose={handleClose}
  title="Título"
>
  Contenido
</Modal>
```

✅ **Composición sobre configuración**
```jsx
// ✅ Flexible con composición
<Dialog>
  <DialogHeader>Título</DialogHeader>
  <DialogBody>Contenido</DialogBody>
  <DialogFooter>
    <Button>Cancelar</Button>
    <Button>Confirmar</Button>
  </DialogFooter>
</Dialog>

// ❌ Rígido con props
<Dialog
  title="Título"
  content="Contenido"
  buttons={['Cancelar', 'Confirmar']}
/>
```

---

## Cuándo dividir componentes

### Señales para dividir:

1. **Más de 200 líneas** → Demasiado grande
2. **Hace múltiples cosas** → Separar responsabilidades
3. **Se repite código** → Extraer común
4. **Difícil de testear** → Simplificar

### Ejemplo: Dividir componente grande

```jsx
// ❌ Componente muy grande
function ProductPage({ productId }) {
  // 50 líneas de lógica...
  // 100 líneas de JSX...
  return (
    <div>
      {/* Header */}
      {/* Galería de imágenes */}
      {/* Info del producto */}
      {/* Reviews */}
      {/* Productos relacionados */}
    </div>
  );
}

// ✅ Dividido en componentes
function ProductPage({ productId }) {
  const product = useProduct(productId);
  
  return (
    <div>
      <ProductHeader product={product} />
      <ProductGallery images={product.images} />
      <ProductInfo product={product} />
      <ProductReviews reviews={product.reviews} />
      <RelatedProducts category={product.category} />
    </div>
  );
}
```

---

## Resumen

✅ **Composición = combinar componentes pequeños**  
✅ **Favorecer composición sobre herencia**  
✅ **children prop para contenido flexible**  
✅ **Slots para múltiples áreas**  
✅ **Compound components para APIs relacionadas**  
✅ **Extraer lógica común**  
✅ **Componentes pequeños y enfocados**

---

## Siguiente paso

Dominas composición. Ahora aprenderás **renderizado condicional** en detalle.

**Siguiente:** [Módulo 09 - Renderizado condicional](./09-condicional-rendering.md)

**Anterior:** [Módulo 07 - Props](./07-props.md)

# Módulo 14: Estado con Objetos y Arrays

## Inmutabilidad en React

En React, **nunca debes modificar el estado directamente**. Siempre debes crear una **copia nueva**.

### ❌ Mutar estado (INCORRECTO)
```jsx
const [user, setUser] = useState({ name: 'Ana', age: 25 });

// ❌ NO HAGAS ESTO
user.age = 26;  // Modifica directamente
setUser(user);  // React no detecta el cambio
```

### ✅ Crear nuevo objeto (CORRECTO)
```jsx
const [user, setUser] = useState({ name: 'Ana', age: 25 });

// ✅ Crea un nuevo objeto
setUser({ ...user, age: 26 });  // React detecta el cambio
```

---

## Spread operator (...)

El **spread operator** copia todas las propiedades de un objeto o array:

### Con objetos
```jsx
const original = { a: 1, b: 2, c: 3 };
const copia = { ...original };  // { a: 1, b: 2, c: 3 }

// Copiar y modificar
const modificado = { ...original, b: 99 };  // { a: 1, b: 99, c: 3 }
```

### Con arrays
```jsx
const original = [1, 2, 3];
const copia = [...original];  // [1, 2, 3]

// Agregar elemento
const conNuevo = [...original, 4];  // [1, 2, 3, 4]
```

---

## Estado con objetos

### Actualizar una propiedad

```jsx
function PerfilUsuario() {
  const [user, setUser] = useState({
    nombre: 'Ana',
    edad: 25,
    email: 'ana@example.com'
  });
  
  const cumplirAnios = () => {
    setUser({
      ...user,        // Copia todas las propiedades
      edad: user.edad + 1  // Sobrescribe solo 'edad'
    });
  };
  
  return (
    <div>
      <p>Nombre: {user.nombre}</p>
      <p>Edad: {user.edad}</p>
      <button onClick={cumplirAnios}>Cumplir años</button>
    </div>
  );
}
```

---

### Actualizar múltiples propiedades

```jsx
function Formulario() {
  const [form, setForm] = useState({
    nombre: '',
    email: '',
    edad: 0
  });
  
  const handleChange = (campo, valor) => {
    setForm({
      ...form,
      [campo]: valor  // Computed property name
    });
  };
  
  return (
    <div>
      <input 
        value={form.nombre}
        onChange={(e) => handleChange('nombre', e.target.value)}
      />
      <input 
        value={form.email}
        onChange={(e) => handleChange('email', e.target.value)}
      />
      <input 
        type="number"
        value={form.edad}
        onChange={(e) => handleChange('edad', Number(e.target.value))}
      />
    </div>
  );
}
```

---

### Patrón común: handleChange genérico

```jsx
function Formulario() {
  const [form, setForm] = useState({
    nombre: '',
    email: '',
    edad: 0
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm({
      ...form,
      [name]: value
    });
  };
  
  return (
    <div>
      <input 
        name="nombre"
        value={form.nombre}
        onChange={handleChange}
      />
      <input 
        name="email"
        value={form.email}
        onChange={handleChange}
      />
      <input 
        name="edad"
        type="number"
        value={form.edad}
        onChange={handleChange}
      />
    </div>
  );
}
```

---

## Objetos anidados

### Actualizar propiedad anidada

```jsx
function PerfilCompleto() {
  const [user, setUser] = useState({
    nombre: 'Ana',
    direccion: {
      calle: 'Main St',
      ciudad: 'Madrid',
      pais: 'España'
    }
  });
  
  const cambiarCiudad = (nuevaCiudad) => {
    setUser({
      ...user,
      direccion: {
        ...user.direccion,  // Copia todo el objeto 'direccion'
        ciudad: nuevaCiudad  // Sobrescribe solo 'ciudad'
      }
    });
  };
  
  return (
    <div>
      <p>Vive en: {user.direccion.ciudad}</p>
      <button onClick={() => cambiarCiudad('Barcelona')}>
        Mudar a Barcelona
      </button>
    </div>
  );
}
```

---

## Estado con arrays

### Agregar elemento al final

```jsx
function ListaTareas() {
  const [tareas, setTareas] = useState(['Tarea 1', 'Tarea 2']);
  
  const agregar = () => {
    setTareas([...tareas, 'Nueva tarea']);
  };
  
  return (
    <div>
      <ul>
        {tareas.map((tarea, i) => <li key={i}>{tarea}</li>)}
      </ul>
      <button onClick={agregar}>Agregar</button>
    </div>
  );
}
```

---

### Agregar elemento al inicio

```jsx
const agregar = () => {
  setTareas(['Nueva tarea', ...tareas]);
};
```

---

### Eliminar elemento por índice

```jsx
function ListaTareas() {
  const [tareas, setTareas] = useState(['Tarea 1', 'Tarea 2', 'Tarea 3']);
  
  const eliminar = (index) => {
    setTareas(tareas.filter((_, i) => i !== index));
  };
  
  return (
    <ul>
      {tareas.map((tarea, index) => (
        <li key={index}>
          {tarea}
          <button onClick={() => eliminar(index)}>❌</button>
        </li>
      ))}
    </ul>
  );
}
```

---

### Modificar elemento por índice

```jsx
function ListaNumeros() {
  const [numeros, setNumeros] = useState([1, 2, 3, 4, 5]);
  
  const duplicar = (index) => {
    setNumeros(numeros.map((num, i) => 
      i === index ? num * 2 : num
    ));
  };
  
  return (
    <ul>
      {numeros.map((num, index) => (
        <li key={index}>
          {num}
          <button onClick={() => duplicar(index)}>x2</button>
        </li>
      ))}
    </ul>
  );
}
```

---

### Reemplazar elemento

```jsx
const reemplazar = (index, nuevoValor) => {
  const copia = [...tareas];
  copia[index] = nuevoValor;
  setTareas(copia);
};

// O con map
const reemplazar = (index, nuevoValor) => {
  setTareas(tareas.map((tarea, i) => 
    i === index ? nuevoValor : tarea
  ));
};
```

---

### Vaciar array

```jsx
const vaciar = () => {
  setTareas([]);
};
```

---

## Arrays de objetos

### Agregar objeto

```jsx
function ListaUsuarios() {
  const [usuarios, setUsuarios] = useState([
    { id: 1, nombre: 'Ana', edad: 25 },
    { id: 2, nombre: 'Luis', edad: 30 }
  ]);
  
  const agregar = () => {
    const nuevoUsuario = {
      id: Date.now(),
      nombre: 'Nuevo',
      edad: 20
    };
    setUsuarios([...usuarios, nuevoUsuario]);
  };
  
  return (
    <div>
      {usuarios.map(user => (
        <div key={user.id}>
          {user.nombre} - {user.edad} años
        </div>
      ))}
      <button onClick={agregar}>Agregar</button>
    </div>
  );
}
```

---

### Eliminar objeto por ID

```jsx
const eliminar = (id) => {
  setUsuarios(usuarios.filter(user => user.id !== id));
};

// Uso
<button onClick={() => eliminar(user.id)}>Eliminar</button>
```

---

### Modificar objeto por ID

```jsx
const cambiarNombre = (id, nuevoNombre) => {
  setUsuarios(usuarios.map(user => 
    user.id === id 
      ? { ...user, nombre: nuevoNombre }
      : user
  ));
};
```

---

### Toggle propiedad booleana

```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, texto: 'Tarea 1', completada: false },
    { id: 2, texto: 'Tarea 2', completada: false }
  ]);
  
  const toggleCompletada = (id) => {
    setTodos(todos.map(todo => 
      todo.id === id 
        ? { ...todo, completada: !todo.completada }
        : todo
    ));
  };
  
  return (
    <ul>
      {todos.map(todo => (
        <li 
          key={todo.id}
          onClick={() => toggleCompletada(todo.id)}
          style={{ 
            textDecoration: todo.completada ? 'line-through' : 'none' 
          }}
        >
          {todo.texto}
        </li>
      ))}
    </ul>
  );
}
```

---

## Ejemplo completo: CRUD de productos

```jsx
function ListaProductos() {
  const [productos, setProductos] = useState([
    { id: 1, nombre: 'Laptop', precio: 1000 },
    { id: 2, nombre: 'Mouse', precio: 25 }
  ]);
  
  const [form, setForm] = useState({ nombre: '', precio: 0 });
  
  // CREATE
  const agregar = () => {
    const nuevo = {
      id: Date.now(),
      nombre: form.nombre,
      precio: form.precio
    };
    setProductos([...productos, nuevo]);
    setForm({ nombre: '', precio: 0 });
  };
  
  // READ (ya está en el render)
  
  // UPDATE
  const actualizar = (id, nuevosDatos) => {
    setProductos(productos.map(p => 
      p.id === id ? { ...p, ...nuevosDatos } : p
    ));
  };
  
  // DELETE
  const eliminar = (id) => {
    setProductos(productos.filter(p => p.id !== id));
  };
  
  return (
    <div>
      {/* Formulario */}
      <div>
        <input 
          placeholder="Nombre"
          value={form.nombre}
          onChange={(e) => setForm({ ...form, nombre: e.target.value })}
        />
        <input 
          type="number"
          placeholder="Precio"
          value={form.precio}
          onChange={(e) => setForm({ ...form, precio: Number(e.target.value) })}
        />
        <button onClick={agregar}>Agregar</button>
      </div>
      
      {/* Lista */}
      <ul>
        {productos.map(producto => (
          <li key={producto.id}>
            {producto.nombre} - ${producto.precio}
            <button onClick={() => eliminar(producto.id)}>❌</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Errores comunes

### ❌ Modificar array directamente

```jsx
// ❌ NO HAGAS ESTO
const agregar = () => {
  tareas.push('Nueva tarea');  // Mutación directa
  setTareas(tareas);  // React no detecta el cambio
};
```

### ✅ Crear nuevo array

```jsx
// ✅ HAZ ESTO
const agregar = () => {
  setTareas([...tareas, 'Nueva tarea']);
};
```

---

### ❌ Modificar objeto directamente

```jsx
// ❌ NO HAGAS ESTO
const cambiar = () => {
  user.nombre = 'Nuevo nombre';  // Mutación directa
  setUser(user);  // React no detecta el cambio
};
```

### ✅ Crear nuevo objeto

```jsx
// ✅ HAZ ESTO
const cambiar = () => {
  setUser({ ...user, nombre: 'Nuevo nombre' });
};
```

---

### ❌ Modificar objeto anidado sin copiar todo

```jsx
// ❌ NO HAGAS ESTO
const cambiar = () => {
  setUser({
    ...user,
    direccion: { ciudad: 'Nueva ciudad' }  // Pierde 'calle' y 'pais'
  });
};
```

### ✅ Copiar objeto anidado completo

```jsx
// ✅ HAZ ESTO
const cambiar = () => {
  setUser({
    ...user,
    direccion: {
      ...user.direccion,  // Copia todas las propiedades
      ciudad: 'Nueva ciudad'
    }
  });
};
```

---

## Métodos de arrays inmutables

| Operación | ❌ Mutable | ✅ Inmutable |
|-----------|-----------|-------------|
| Agregar al final | `arr.push(item)` | `[...arr, item]` |
| Agregar al inicio | `arr.unshift(item)` | `[item, ...arr]` |
| Eliminar | `arr.splice(index, 1)` | `arr.filter((_, i) => i !== index)` |
| Reemplazar | `arr[index] = nuevo` | `arr.map((item, i) => i === index ? nuevo : item)` |
| Ordenar | `arr.sort()` | `[...arr].sort()` |
| Invertir | `arr.reverse()` | `[...arr].reverse()` |

---

## Estado complejo: carrito de compras

```jsx
function CarritoCompras() {
  const [carrito, setCarrito] = useState({
    items: [],
    total: 0
  });
  
  const agregarItem = (producto) => {
    const nuevoItem = {
      id: producto.id,
      nombre: producto.nombre,
      precio: producto.precio,
      cantidad: 1
    };
    
    setCarrito({
      items: [...carrito.items, nuevoItem],
      total: carrito.total + producto.precio
    });
  };
  
  const eliminarItem = (id) => {
    const item = carrito.items.find(i => i.id === id);
    
    setCarrito({
      items: carrito.items.filter(i => i.id !== id),
      total: carrito.total - (item.precio * item.cantidad)
    });
  };
  
  const cambiarCantidad = (id, cantidad) => {
    const itemAnterior = carrito.items.find(i => i.id === id);
    const diferencia = (cantidad - itemAnterior.cantidad) * itemAnterior.precio;
    
    setCarrito({
      items: carrito.items.map(item =>
        item.id === id
          ? { ...item, cantidad }
          : item
      ),
      total: carrito.total + diferencia
    });
  };
  
  return (
    <div>
      <h2>Carrito (${carrito.total})</h2>
      <ul>
        {carrito.items.map(item => (
          <li key={item.id}>
            {item.nombre} - ${item.precio} x {item.cantidad}
            <button onClick={() => cambiarCantidad(item.id, item.cantidad + 1)}>
              +
            </button>
            <button onClick={() => cambiarCantidad(item.id, item.cantidad - 1)}>
              -
            </button>
            <button onClick={() => eliminarItem(item.id)}>
              ❌
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Ejercicio 1: Lista de tareas con edición

Crea una lista de tareas donde puedas:
- Agregar tarea
- Marcar como completada
- Eliminar tarea
- Editar el texto de una tarea

```jsx
function TodoListCompleta() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  // Implementa las funciones:
  // - agregar()
  // - toggleCompletada(id)
  // - eliminar(id)
  // - editarTexto(id, nuevoTexto)
}
```

<details>
<summary>Ver solución</summary>

```jsx
function TodoListCompleta() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const agregar = () => {
    if (input.trim()) {
      const nuevo = {
        id: Date.now(),
        texto: input,
        completada: false
      };
      setTodos([...todos, nuevo]);
      setInput('');
    }
  };
  
  const toggleCompletada = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id
        ? { ...todo, completada: !todo.completada }
        : todo
    ));
  };
  
  const eliminar = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  const editarTexto = (id, nuevoTexto) => {
    setTodos(todos.map(todo =>
      todo.id === id
        ? { ...todo, texto: nuevoTexto }
        : todo
    ));
  };
  
  return (
    <div>
      <input 
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyDown={(e) => e.key === 'Enter' && agregar()}
      />
      <button onClick={agregar}>Agregar</button>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input 
              type="checkbox"
              checked={todo.completada}
              onChange={() => toggleCompletada(todo.id)}
            />
            <input 
              value={todo.texto}
              onChange={(e) => editarTexto(todo.id, e.target.value)}
              style={{
                textDecoration: todo.completada ? 'line-through' : 'none'
              }}
            />
            <button onClick={() => eliminar(todo.id)}>❌</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```
</details>

---

## Ejercicio 2: Formulario de perfil

Crea un formulario que maneje un objeto de usuario con:
- nombre
- email
- teléfono
- dirección (objeto con calle, ciudad, país)

```jsx
function FormularioPerfil() {
  const [user, setUser] = useState({
    nombre: '',
    email: '',
    telefono: '',
    direccion: {
      calle: '',
      ciudad: '',
      pais: ''
    }
  });
  
  // Implementa handleChange para propiedades normales
  // y handleDireccionChange para propiedades de dirección
}
```

<details>
<summary>Ver solución</summary>

```jsx
function FormularioPerfil() {
  const [user, setUser] = useState({
    nombre: '',
    email: '',
    telefono: '',
    direccion: {
      calle: '',
      ciudad: '',
      pais: ''
    }
  });
  
  const handleChange = (e) => {
    setUser({
      ...user,
      [e.target.name]: e.target.value
    });
  };
  
  const handleDireccionChange = (e) => {
    setUser({
      ...user,
      direccion: {
        ...user.direccion,
        [e.target.name]: e.target.value
      }
    });
  };
  
  return (
    <form>
      <input 
        name="nombre"
        value={user.nombre}
        onChange={handleChange}
        placeholder="Nombre"
      />
      <input 
        name="email"
        value={user.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input 
        name="telefono"
        value={user.telefono}
        onChange={handleChange}
        placeholder="Teléfono"
      />
      
      <h3>Dirección</h3>
      <input 
        name="calle"
        value={user.direccion.calle}
        onChange={handleDireccionChange}
        placeholder="Calle"
      />
      <input 
        name="ciudad"
        value={user.direccion.ciudad}
        onChange={handleDireccionChange}
        placeholder="Ciudad"
      />
      <input 
        name="pais"
        value={user.direccion.pais}
        onChange={handleDireccionChange}
        placeholder="País"
      />
      
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </form>
  );
}
```
</details>

---

## Resumen

✅ **Nunca modifiques el estado directamente**  
✅ **Usa spread operator** para copiar objetos y arrays  
✅ **Objetos:** `{ ...obj, propiedad: nuevoValor }`  
✅ **Arrays agregar:** `[...arr, nuevo]`  
✅ **Arrays eliminar:** `arr.filter(item => condicion)`  
✅ **Arrays modificar:** `arr.map(item => condicion ? nuevo : item)`  
✅ **Objetos anidados:** copia cada nivel  
✅ **Arrays de objetos:** usa `map` con spread

---

## Siguiente paso

Ya sabes trabajar con objetos y arrays en el estado. Ahora aprenderás **cómo compartir estado entre componentes**.

**Siguiente:** [Módulo 15 - Levantar estado](./15-levantar-estado.md)

**Anterior:** [Módulo 13 - Eventos en React](./13-eventos-react.md)

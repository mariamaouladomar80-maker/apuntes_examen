# 📄 APUNTES RECICLABLES — ZUSTAND (Estado Global en React)
## Examen Práctico | Basado en teoría clase + Parcial 1845

> 🎯 **Objetivo**: Plantillas listas para copiar, pegar y cambiar 2-3 nombres.
> **Filosofía**: El 80% del código de Zustand es SIEMPRE IGUAL.
> **En el examen**: Abre esto, copia el bloque, adapta nombres de tabla/campos, ejecuta.

---

## ✅ CHECKLIST DEL EXAMEN (Tarea de Zustand)

- [ ] 1. Crear store con `create()` y estado inicial
- [ ] 2. Acciones que usan `set()` con función (no objeto) cuando dependen del estado anterior
- [ ] 3. Usar el store en componente con suscripción selectiva
- [ ] 4. (Si pide persistencia) Añadir `persist` middleware con `name`
- [ ] 5. Mostrar contador/total en header o badge
- [ ] 6. Demostrar que funciona: añadir items, ver que se actualiza la UI

---

## 🧩 REGLA DE ORO: ¿QUÉ CAMBIA Y QUÉ ES FIJO?

| Parte | ¿Cambia? | Ejemplo fijo |
|-------|----------|--------------|
| Nombre del store | ✅ SIEMPRE | `useCarritoStore`, `useFavStore`... |
| Nombre del estado | ✅ SIEMPRE | `items`, `productos`, `favoritos`... |
| Nombre de las acciones | ✅ SIEMPRE | `agregar`, `eliminar`, `vaciar`... |
| Estructura `create((set) => ({...}))` | ❌ NO | Siempre igual |
| `set((estado) => ({...}))` con función | ❌ NO | Siempre para añadir/eliminar/modificar |
| `set({ campo: valor })` con objeto | ⚠️ A veces | Solo para resetear o asignar directo |
| Spread `[...estado.items, item]` | ❌ NO | Siempre para añadir al array |
| `.filter((i) => i.id !== id)` | ❌ NO | Siempre para eliminar del array |
| `.map((i) => i.id === id ? {...} : i)` | ❌ NO | Siempre para modificar un elemento |
| Suscripción selectiva `(estado) => estado.x` | ❌ NO | Siempre igual estructura |
| `persist` middleware | ⚠️ Si pide persistencia | Siempre `persist((set) => ({...}), {name: '...'})` |
| `partialize` | ⚠️ Si pide persistencia | Para excluir funciones del localStorage |

---

## 🚀 PLANTILLA 1: STORE BÁSICO (El de clase)

### Store
```js
// store/useCarritoStore.js
import { create } from 'zustand'

export const useCarritoStore = create((set) => ({
  // ═══════════════════════════════════════════════════════════
  // ESTADO INICIAL (CAMBIA: nombre del estado y valor inicial)
  // ═══════════════════════════════════════════════════════════
  items: [],

  // ═══════════════════════════════════════════════════════════
  // ACCIONES (CAMBIA: nombres de las acciones y lógica)
  // ═══════════════════════════════════════════════════════════

  // Añadir item al final del array
  agregar: (item) => set((estado) => ({
    items: [...estado.items, item],
  })),

  // Eliminar item por id
  eliminar: (id) => set((estado) => ({
    items: estado.items.filter((i) => i.id !== id),
  })),

  // Vaciar el array
  vaciar: () => set({ items: [] }),
}))
```

**¿Qué cambia?**
- `items` → `productos`, `favoritos`, `cursos`...
- `agregar`, `eliminar`, `vaciar` → nombres que pida el profe
- El campo por el que filtras (`i.id`) → podría ser `i.id_producto`, etc.

**¿Qué NUNCA cambia?**
- `import { create } from 'zustand'`
- `export const useXStore = create((set) => ({...}))`
- `set((estado) => ({...}))` con función para operaciones que dependen del estado anterior
- `[...estado.items, item]` para añadir (inmutable)
- `estado.items.filter((i) => i.id !== id)` para eliminar (inmutable)

---

### Componente que usa el store
```jsx
// components/BotonAnadir.jsx
'use client'

import { useCarritoStore } from '@/store/useCarritoStore'

export default function BotonAnadir({ producto }) {
  // ═══════════════════════════════════════════════════════════
  // SUSCRIPCIÓN SELECTIVA (NUNCA CAMBIA la estructura)
  // ═══════════════════════════════════════════════════════════

  // Opción A: Suscribirse solo a la acción (NO re-renderiza al cambiar items)
  const agregar = useCarritoStore((estado) => estado.agregar)

  // Opción B: Suscribirse al array (SI re-renderiza cuando cambia items)
  const items = useCarritoStore((estado) => estado.items)

  // Opción C: Suscribirse al contador (SI re-renderiza solo cuando cambia la cuenta)
  const total = useCarritoStore((estado) => estado.items.length)

  return (
    <button
      onClick={() => agregar(producto)}
      className="bg-blue-500 text-white px-4 py-2 rounded"
    >
      Añadir al carrito ({total})
    </button>
  )
}
```

**¿Qué cambia?**
- Nombre del store (`useCarritoStore`)
- Nombre de la acción (`agregar`)
- Nombre del estado (`items`)
- Props del componente (`producto`)

**¿Qué NUNCA cambia?**
- `'use client'` al inicio
- `useXStore((estado) => estado.algo)` selector
- `onClick={() => agregar(producto)}`

---

## 🚀 PLANTILLA 2: STORE CON PERSISTENCIA (Si pide que se recuerde al recargar)

```js
// store/useCarritoStore.js
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useCarritoStore = create(
  persist(
    (set) => ({
      // Estado
      items: [],

      // Acciones
      agregar: (item) => set((estado) => ({
        items: [...estado.items, item],
      })),

      eliminar: (id) => set((estado) => ({
        items: estado.items.filter((i) => i.id !== id),
      })),

      vaciar: () => set({ items: [] }),
    }),
    {
      // ═══════════════════════════════════════════════════════════
      // CONFIGURACIÓN PERSIST (CAMBIA: nombre de la clave)
      // ═══════════════════════════════════════════════════════════
      name: 'carrito-storage',  // nombre en localStorage
    }
  )
)
```

**¿Qué cambia?**
- `name: 'carrito-storage'` → cualquier nombre identificativo

**¿Qué NUNCA cambia?**
- `import { persist } from 'zustand/middleware'`
- `persist((set) => ({...}), { name: '...' })` estructura

---

## 🚀 PLANTILLA 3: CARRITO CON CANTIDAD (Producto repetido = sumar cantidad)

**Cuándo usar**: Si el profe pide "añadir al carrito" y que si el producto ya existe, sume 1 a la cantidad en vez de duplicar.

```js
// store/useCarritoStore.js
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useCarritoStore = create(
  persist(
    (set) => ({
      items: [],

      // Añadir: si existe, suma cantidad. Si no, lo añade con cantidad 1
      agregar: (producto) => set((estado) => {
        const existe = estado.items.find((i) => i.id === producto.id)

        if (existe) {
          return {
            items: estado.items.map((i) =>
              i.id === producto.id
                ? { ...i, cantidad: i.cantidad + 1 }
                : i
            ),
          }
        }

        return {
          items: [...estado.items, { ...producto, cantidad: 1 }],
        }
      }),

      // Quitar 1 unidad. Si cantidad llega a 0, eliminar del array
      quitar: (id) => set((estado) => ({
        items: estado.items
          .map((i) => (i.id === id ? { ...i, cantidad: i.cantidad - 1 } : i))
          .filter((i) => i.cantidad > 0),
      })),

      // Eliminar completamente un producto
      eliminar: (id) => set((estado) => ({
        items: estado.items.filter((i) => i.id !== id),
      })),

      vaciar: () => set({ items: [] }),

      // Total de unidades (para badge)
      getTotalItems: () => {
        // Esto se usa fuera del store o con getState()
      },
    }),
    {
      name: 'carrito-storage',
    }
  )
)
```

### Componente con cantidad y subtotal
```jsx
// components/ItemCarrito.jsx
'use client'

import { useCarritoStore } from '@/store/useCarritoStore'

export default function ItemCarrito({ item }) {
  const quitar = useCarritoStore((estado) => estado.quitar)
  const agregar = useCarritoStore((estado) => estado.agregar)
  const eliminar = useCarritoStore((estado) => estado.eliminar)

  return (
    <div className="flex justify-between items-center border p-4 rounded mb-2">
      <div>
        <h3 className="font-bold">{item.nombre}</h3>
        <p className="text-gray-600">{item.precio}€ x {item.cantidad}</p>
        <p className="font-semibold">Subtotal: {item.precio * item.cantidad}€</p>
      </div>
      <div className="flex gap-2">
        <button onClick={() => quitar(item.id)} className="bg-gray-300 px-3 py-1 rounded">-</button>
        <span className="px-2">{item.cantidad}</span>
        <button onClick={() => agregar(item)} className="bg-gray-300 px-3 py-1 rounded">+</button>
        <button onClick={() => eliminar(item.id)} className="bg-red-500 text-white px-3 py-1 rounded">🗑️</button>
      </div>
    </div>
  )
}
```

---

## 🚀 PLANTILLA 4: HEADER CON CONTADOR (Badge del carrito)

```jsx
// components/Header.jsx
'use client'

import { useCarritoStore } from '@/store/useCarritoStore'

export default function Header() {
  // Suscribirse solo al número total (re-renderiza solo cuando cambia la cuenta)
  const totalItems = useCarritoStore((estado) =>
    estado.items.reduce((acc, item) => acc + (item.cantidad || 1), 0)
  )

  return (
    <header className="bg-gray-800 text-white p-4 flex justify-between items-center">
      <h1 className="text-xl font-bold">Mi Tienda</h1>
      <div className="relative">
        <span className="text-2xl">🛒</span>
        {totalItems > 0 && (
          <span className="absolute -top-2 -right-2 bg-red-500 text-white rounded-full px-2 py-0.5 text-sm">
            {totalItems}
          </span>
        )}
      </div>
    </header>
  )
}
```

**Concepto clave**: `reduce` suma las cantidades. Si el item no tiene `cantidad`, usa `1` por defecto (`item.cantidad || 1`).

---

## 🚀 PLANTILLA 5: LISTA DE PRODUCTOS + BOTÓN AÑADIR (Integración con API/Supabase)

**Cuándo usar**: Si el profe pide "mostrar productos de la base de datos y añadirlos al carrito con Zustand".

```jsx
// app/productos/page.jsx
'use client'

import { useState, useEffect } from 'react'
import { useCarritoStore } from '@/store/useCarritoStore'
import { supabase } from '@/lib/supabase'

export default function ProductosPage() {
  const [productos, setProductos] = useState([])
  const agregar = useCarritoStore((estado) => estado.agregar)

  useEffect(() => {
    const cargarProductos = async () => {
      const { data } = await supabase.from('productos').select('*')
      setProductos(data || [])
    }
    cargarProductos()
  }, [])

  return (
    <main className="p-8">
      <h1 className="text-3xl font-bold mb-6">Productos</h1>
      <div className="grid grid-cols-3 gap-4">
        {productos.map((producto) => (
          <div key={producto.id} className="border rounded-lg p-4 shadow">
            <h2 className="font-bold text-lg">{producto.nombre}</h2>
            <p className="text-green-600 font-bold">{producto.precio}€</p>
            <button
              onClick={() => agregar(producto)}
              className="mt-2 w-full bg-blue-500 text-white py-2 rounded"
            >
              Añadir al carrito
            </button>
          </div>
        ))}
      </div>
    </main>
  )
}
```

**¿Qué cambia?**
- Tabla de Supabase (`productos`)
- Campos del producto (`nombre`, `precio`)
- Diseño (clases Tailwind)

**¿Qué NUNCA cambia?**
- `useState([])` + `useEffect(() => {...}, [])` para cargar datos
- `useCarritoStore((estado) => estado.agregar)` para la acción
- `onClick={() => agregar(producto)}`

---

## 🔥 VARIANTES QUE PUEDE PEDIR EL PROFE

### Variante A: Favoritos (toggle en vez de agregar/eliminar)

```js
// Store de favoritos
export const useFavStore = create((set) => ({
  favoritos: [],

  toggleFav: (producto) => set((estado) => {
    const existe = estado.favoritos.find((f) => f.id === producto.id)
    if (existe) {
      return { favoritos: estado.favoritos.filter((f) => f.id !== producto.id) }
    }
    return { favoritos: [...estado.favoritos, producto] }
  }),
}))
```

---

### Variante B: Store con múltiples estados (items + filtros + orden)

```js
export const useAppStore = create((set) => ({
  // Estado
  items: [],
  filtro: 'todos',
  orden: 'nombre',

  // Acciones
  setFiltro: (filtro) => set({ filtro }),
  setOrden: (orden) => set({ orden }),
  agregar: (item) => set((estado) => ({ items: [...estado.items, item] })),

  // Valor computado (filtrado + ordenado)
  getItemsFiltrados: () => {
    const { items, filtro, orden } = useAppStore.getState()
    let resultado = [...items]
    if (filtro !== 'todos') resultado = resultado.filter((i) => i.categoria === filtro)
    if (orden === 'nombre') resultado.sort((a, b) => a.nombre.localeCompare(b.nombre))
    return resultado
  },
}))
```

---

### Variante C: Partialize (persistir solo ciertos campos)

```js
persist(
  (set) => ({
    items: [],
    usuario: null,
    cargando: false,
    agregar: (item) => set((s) => ({ items: [...s.items, item] })),
  }),
  {
    name: 'mi-store',
    partialize: (estado) => ({
      items: estado.items,      // persistir
      usuario: estado.usuario,  // persistir
      // cargando: NO persistir (estado efímero)
      // agregar: NO persistir (función, no serializable)
    }),
  }
)
```

---

## ⚡ CHEAT SHEET ULTRA-RÁPIDO (Copiar y pegar en el examen)

### Store mínimo
```js
import { create } from 'zustand'
export const useXStore = create((set) => ({
  items: [],
  agregar: (item) => set((s) => ({ items: [...s.items, item] })),
  eliminar: (id) => set((s) => ({ items: s.items.filter((i) => i.id !== id) })),
  vaciar: () => set({ items: [] }),
}))
```

### Store con persistencia
```js
import { create } from 'zustand'
import { persist } from 'zustand/middleware'
export const useXStore = create(persist((set) => ({
  items: [],
  agregar: (item) => set((s) => ({ items: [...s.items, item] })),
  eliminar: (id) => set((s) => ({ items: s.items.filter((i) => i.id !== id) })),
}), { name: 'storage' }))
```

### Carrito con cantidad
```js
agregar: (p) => set((s) => {
  const existe = s.items.find((i) => i.id === p.id)
  if (existe) return { items: s.items.map((i) => i.id === p.id ? { ...i, cantidad: i.cantidad + 1 } : i) }
  return { items: [...s.items, { ...p, cantidad: 1 }] }
}),
quitar: (id) => set((s) => ({ items: s.items.map((i) => i.id === id ? { ...i, cantidad: i.cantidad - 1 } : i).filter((i) => i.cantidad > 0) })),
```

### Componente que usa el store
```jsx
'use client'
import { useXStore } from '@/store/useXStore'
export default function Boton({ item }) {
  const agregar = useXStore((s) => s.agregar)
  const total = useXStore((s) => s.items.length)
  return <button onClick={() => agregar(item)}>Añadir ({total})</button>
}
```

### Header con contador
```jsx
const total = useXStore((s) => s.items.reduce((acc, i) => acc + (i.cantidad || 1), 0))
```

---

## 🐛 ERRORES COMUNES Y SOLUCIONES RÁPIDAS

| Error | Por qué pasa | Solución en 5 segundos |
|-------|-------------|------------------------|
| `items` no se actualiza en la UI | Mutaste el array directamente | Usa `[...estado.items, item]` en vez de `push` |
| Componente no re-renderiza | Te suscribiste a la acción, no al estado | Si necesitas ver el estado, suscríbete a `estado.items` |
| Re-renderiza todo el árbol | Suscripción total `useXStore()` sin selector | Usa `useXStore((s) => s.campo)` selector selectivo |
| `persist` no guarda funciones | Las funciones no son serializables en JSON | Usa `partialize` para excluir funciones |
| `localStorage` no definido | Estás en servidor (SSR) | Zustand maneja esto solo, pero asegúrate de usar `'use client'` |
| `cantidad` es undefined | El item no tenía campo cantidad al añadir | Añade `{ ...producto, cantidad: 1 }` al insertar |
| `reduce` da NaN | Algún item no tiene cantidad | Usa `(i.cantidad || 1)` como fallback |
| Store vacío al recargar | Olvidaste `persist` o `name` | Añade `persist(..., { name: 'clave' })` |

---

## ✅ CHECKLIST FINAL ANTES DE ENTREGAR (Zustand)

- [ ] Store creado con `create((set) => ({...}))`
- [ ] Estado inicial definido (array vacío, objeto, etc.)
- [ ] Acciones que usan `set()` con función cuando dependen del estado anterior
- [ ] Operaciones inmutables: spread `[...]`, `.filter()`, `.map()`
- [ ] Componentes con `'use client'` si usan hooks o eventos
- [ ] Suscripción selectiva: `useStore((s) => s.campo)` en vez de `useStore()`
- [ ] Si hay persistencia: `persist` middleware importado y `name` definido
- [ ] Si hay persistencia: `partialize` para excluir funciones si es necesario
- [ ] Badge/contador en header que se actualiza en tiempo real
- [ ] Demostración de que funciona: añadir items, ver que se refleja en la UI

---

## 📝 EJEMPLOS DE PREGUNTAS TIPO EXAMEN (con respuestas listas)

### Pregunta 1: "Crea un store Zustand con items, agregar y vaciar. Luego un botón que lo use."

**Respuesta completa lista para copiar:**

```js
// store/useCarritoStore.js
import { create } from 'zustand'

export const useCarritoStore = create((set) => ({
  items: [],
  agregar: (item) => set((s) => ({ items: [...s.items, item] })),
  vaciar: () => set({ items: [] }),
}))
```

```jsx
// components/BotonAnadir.jsx
'use client'
import { useCarritoStore } from '@/store/useCarritoStore'

export default function BotonAnadir({ producto }) {
  const agregar = useCarritoStore((s) => s.agregar)
  return (
    <button onClick={() => agregar(producto)} className="bg-blue-500 text-white px-4 py-2 rounded">
      Añadir {producto.nombre}
    </button>
  )
}
```

---

### Pregunta 2: "Añade persistencia al carrito para que se recuerde al recargar la página."

**Respuesta completa lista para copiar:**

```js
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useCarritoStore = create(
  persist(
    (set) => ({
      items: [],
      agregar: (item) => set((s) => ({ items: [...s.items, item] })),
      eliminar: (id) => set((s) => ({ items: s.items.filter((i) => i.id !== id) })),
      vaciar: () => set({ items: [] }),
    }),
    { name: 'carrito-storage' }
  )
)
```

---

### Pregunta 3: "Crea un carrito donde si el producto ya existe, sume 1 a la cantidad. Muestra el total de items en el header."

**Respuesta completa lista para copiar:**

```js
// store/useCarritoStore.js
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useCarritoStore = create(
  persist(
    (set) => ({
      items: [],

      agregar: (producto) => set((s) => {
        const existe = s.items.find((i) => i.id === producto.id)
        if (existe) {
          return { items: s.items.map((i) => i.id === producto.id ? { ...i, cantidad: i.cantidad + 1 } : i) }
        }
        return { items: [...s.items, { ...producto, cantidad: 1 }] }
      }),

      quitar: (id) => set((s) => ({
        items: s.items.map((i) => i.id === id ? { ...i, cantidad: i.cantidad - 1 } : i).filter((i) => i.cantidad > 0)
      })),

      vaciar: () => set({ items: [] }),
    }),
    { name: 'carrito-storage' }
  )
)
```

```jsx
// components/Header.jsx
'use client'
import { useCarritoStore } from '@/store/useCarritoStore'

export default function Header() {
  const total = useCarritoStore((s) => s.items.reduce((acc, i) => acc + i.cantidad, 0))
  return (
    <header className="bg-gray-800 text-white p-4 flex justify-between">
      <h1>Mi Tienda</h1>
      <span>🛒 {total} items</span>
    </header>
  )
}
```

---

> 💡 **Tip final**: En el examen, lee si pide persistencia o no. Si no la pide, no la añadas (es código extra innecesario). Si pide "añadir al carrito", usa la Plantilla 3 (con cantidad). Si solo pide "lista de favoritos", usa la Plantilla 1 (array simple). ¡No te compliques!

# 📄 01 — CRUD COMPLETO EN NEXT.JS + SUPABASE
## Plantilla reciclable: SELECT, INSERT, UPDATE, DELETE

> 🎯 **Solo cambias**: nombre de tabla y campos del formulario
> **Todo lo demás**: copiar y pegar tal cual

---

## 🔴 PARTE FIJA (nunca cambia)

### Cliente Supabase (`utils/supabase/client.js`)

```javascript
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY

export const supabase = createClient(supabaseUrl, supabaseKey)
```

### Variables de entorno (`.env.local`)

```env
NEXT_PUBLIC_SUPABASE_URL=https://tu-proyecto.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=tu-anon-key
```

---

## 🟡 PARTE QUE CAMBIA (adaptar según enunciado)

### SQL para crear tabla

```sql
-- Cambia: nombre de tabla y campos
CREATE TABLE IF NOT EXISTS cursos (     -- ← CAMBIA: cursos, productos, libros...
  id         uuid DEFAULT gen_random_uuid() PRIMARY KEY,  -- ← FIJO
  nombre     text NOT NULL,                               -- ← CAMBIA
  modalidad  text NOT NULL,                               -- ← CAMBIA
  horas      integer NOT NULL,                            -- ← CAMBIA
  creado_en  timestamptz DEFAULT now()                    -- ← FIJO
);
```

### Página principal CRUD (`app/page.jsx`)

```jsx
"use client"

import { useState, useEffect } from "react"
import { supabase } from "@/utils/supabase/client"

// ═══════════════════════════════════════════════════════════
// CONFIGURACIÓN: SOLO CAMBIAR ESTAS 2 LÍNEAS
// ═══════════════════════════════════════════════════════════
const TABLA = "cursos"                    // ← CAMBIA: nombre de tabla
const CAMPOS = ["nombre", "modalidad", "horas"]  // ← CAMBIA: campos

export default function Home() {
  const [lista, setLista] = useState([])

  // Estados del formulario (uno por cada campo)
  const [nombre, setNombre] = useState("")
  const [modalidad, setModalidad] = useState("online")
  const [horas, setHoras] = useState("")

  // Estados para edición
  const [editando, setEditando] = useState(null)
  const [nombreEdit, setNombreEdit] = useState("")
  const [modalidadEdit, setModalidadEdit] = useState("")
  const [horasEdit, setHorasEdit] = useState("")

  // ═══════════════════════════════════════════════════════════
  // SELECT: cargar datos (SIEMPRE IGUAL)
  // ═══════════════════════════════════════════════════════════
  const cargarDatos = async () => {
    const { data, error } = await supabase
      .from(TABLA)
      .select("*")
      .order("creado_en", { ascending: false })

    if (error) console.error("Error:", error)
    else setLista(data || [])
  }

  useEffect(() => {
    cargarDatos()
  }, [])

  // ═══════════════════════════════════════════════════════════
  // INSERT: crear nuevo (SIEMPRE IGUAL, cambia objeto)
  // ═══════════════════════════════════════════════════════════
  const guardar = async (e) => {
    e.preventDefault()

    const { data, error } = await supabase
      .from(TABLA)
      .insert({
        nombre: nombre,           // ← CAMBIA según campos
        modalidad: modalidad,     // ← CAMBIA
        horas: parseInt(horas)    // ← CAMBIA (parseInt, parseFloat, etc.)
      })
      .select()

    if (error) {
      alert("Error: " + error.message)
      return
    }

    // Limpiar formulario
    setNombre("")
    setModalidad("online")
    setHoras("")
    cargarDatos()
  }

  // ═══════════════════════════════════════════════════════════
  // UPDATE: editar (SIEMPRE IGUAL, cambia objeto)
  // ═══════════════════════════════════════════════════════════
  const guardarEdicion = async (id) => {
    const { error } = await supabase
      .from(TABLA)
      .update({
        nombre: nombreEdit,
        modalidad: modalidadEdit,
        horas: parseInt(horasEdit)
      })
      .eq("id", id)

    if (error) {
      alert("Error al editar: " + error.message)
      return
    }

    setEditando(null)
    cargarDatos()
  }

  // ═══════════════════════════════════════════════════════════
  // DELETE: eliminar (SIEMPRE IGUAL)
  // ═══════════════════════════════════════════════════════════
  const eliminar = async (id) => {
    if (!confirm("¿Eliminar?")) return

    const { error } = await supabase
      .from(TABLA)
      .delete()
      .eq("id", id)

    if (error) alert("Error al eliminar")
    else cargarDatos()
  }

  // ═══════════════════════════════════════════════════════════
  // HTML: formulario + lista (CAMBIA inputs y columnas)
  // ═══════════════════════════════════════════════════════════
  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">Gestión de {TABLA}</h1>

      {/* ─── FORMULARIO INSERT ─── */}
      <form onSubmit={guardar} className="bg-gray-100 p-4 rounded mb-6 space-y-3">
        <h2 className="font-bold">Nuevo</h2>

        <input
          type="text"
          placeholder="Nombre"
          value={nombre}
          onChange={(e) => setNombre(e.target.value)}
          className="w-full border p-2 rounded"
          required
        />

        <select
          value={modalidad}
          onChange={(e) => setModalidad(e.target.value)}
          className="w-full border p-2 rounded"
        >
          <option value="online">Online</option>
          <option value="presencial">Presencial</option>
        </select>

        <input
          type="number"
          placeholder="Horas"
          value={horas}
          onChange={(e) => setHoras(e.target.value)}
          className="w-full border p-2 rounded"
          required
        />

        <button className="w-full bg-blue-600 text-white p-2 rounded hover:bg-blue-700">
          Guardar
        </button>
      </form>

      {/* ─── LISTA CON EDITAR Y ELIMINAR ─── */}
      <div className="grid grid-cols-3 gap-4">
        {lista.map((item) => (
          <div key={item.id} className="border p-3 rounded bg-white">

            {editando === item.id ? (
              // MODO EDICIÓN
              <div className="space-y-2">
                <input
                  value={nombreEdit}
                  onChange={(e) => setNombreEdit(e.target.value)}
                  className="w-full border p-1 rounded"
                />
                <select
                  value={modalidadEdit}
                  onChange={(e) => setModalidadEdit(e.target.value)}
                  className="w-full border p-1 rounded"
                >
                  <option value="online">Online</option>
                  <option value="presencial">Presencial</option>
                </select>
                <input
                  type="number"
                  value={horasEdit}
                  onChange={(e) => setHorasEdit(e.target.value)}
                  className="w-full border p-1 rounded"
                />
                <div className="flex gap-2">
                  <button 
                    onClick={() => guardarEdicion(item.id)}
                    className="flex-1 bg-green-500 text-white p-1 rounded text-sm"
                  >
                    Guardar
                  </button>
                  <button 
                    onClick={() => setEditando(null)}
                    className="flex-1 bg-gray-500 text-white p-1 rounded text-sm"
                  >
                    Cancelar
                  </button>
                </div>
              </div>
            ) : (
              // MODO LECTURA
              <>
                <h3 className="font-bold">{item.nombre}</h3>
                <p className="text-blue-600">{item.modalidad}</p>
                <p className="text-gray-500">{item.horas} horas</p>

                <div className="flex gap-2 mt-2">
                  <button 
                    onClick={() => {
                      setEditando(item.id)
                      setNombreEdit(item.nombre)
                      setModalidadEdit(item.modalidad)
                      setHorasEdit(item.horas)
                    }}
                    className="flex-1 bg-yellow-500 text-white p-1 rounded text-sm"
                  >
                    Editar
                  </button>
                  <button 
                    onClick={() => eliminar(item.id)}
                    className="flex-1 bg-red-500 text-white p-1 rounded text-sm"
                  >
                    Eliminar
                  </button>
                </div>
              </>
            )}
          </div>
        ))}
      </div>
    </div>
  )
}
```

---

## ✏️ CÓMO ADAPTAR EN 30 SEGUNDOS

### Cambio 1: Nombre de tabla
```javascript
// Antes:
const TABLA = "cursos"
// Después:
const TABLA = "productos"    // o "libros", "empleados"...
```

### Cambio 2: Estados del formulario
```javascript
// Antes (3 campos):
const [nombre, setNombre] = useState("")
const [modalidad, setModalidad] = useState("online")
const [horas, setHoras] = useState("")

// Después (2 campos):
const [nombre, setNombre] = useState("")
const [precio, setPrecio] = useState("")
```

### Cambio 3: Objeto insert/update
```javascript
// Antes:
{ nombre: nombre, modalidad: modalidad, horas: parseInt(horas) }

// Después:
{ nombre: nombre, precio: parseFloat(precio) }
```

### Cambio 4: Inputs del formulario
```jsx
// Cambia los <input> y <select> para que coincidan con los nuevos campos
```

### Cambio 5: Estados de edición
```javascript
// Añade/quita estados según campos:
const [nombreEdit, setNombreEdit] = useState("")
const [precioEdit, setPrecioEdit] = useState("")
```

> ⚠️ **Lo que NUNCA cambias**: `useEffect`, `cargarDatos`, `.select('*')`, `.insert()`, `.update()`, `.delete().eq('id', id)`, `'use client'`.

---

## 🐛 ERRORES COMUNES

| Error | Solución |
|-------|----------|
| `supabaseKey is required` | Revisar `.env.local` y reiniciar `npm run dev` |
| `useState is not defined` | `import { useState, useEffect } from "react"` |
| No aparecen datos | RLS activado → desactivar en Supabase Studio |
| `insert` falla | Revisar que nombres de campos coincidan EXACTAMENTE |
| `delete` no borra | Comprobar que `item.id` existe |

---

## ✅ CHECKLIST CRUD

- [ ] `'use client'` en primera línea
- [ ] `useEffect` carga datos al montar
- [ ] `useState` para lista y campos del formulario
- [ ] Formulario con `onSubmit` → `.insert()`
- [ ] Botón Editar → `.update()`
- [ ] Botón Eliminar → `.delete().eq('id', ...)`
- [ ] Después de insert/update/delete → recargar lista

# 📦 APUNTE 00 — INSTALACIÓN COMPLETA PASO A PASO
## Next.js + Supabase Local (Docker) + Zustand

> 🎯 **Objetivo**: Tener el proyecto listo para programar en 10 minutos.
> Guarda este archivo en: `tu-proyecto/document/00-instalacion.md`
> **En el examen**: Ábrelo, sigue los pasos numerados, copia y pega.

---

## ✅ CHECKLIST RÁPIDA (para marcar con el ratón en el examen)

- [ ] 1. Encender Docker Desktop
- [ ] 2. Crear proyecto Next.js (con carpeta `src/`)
- [ ] 3. Instalar dependencias (`@supabase/supabase-js`, `@supabase/ssr`, `zustand`)
- [ ] 4. Inicializar Supabase local (`npx supabase init`)
- [ ] 5. Arrancar Supabase local (`npx supabase start`)
- [ ] 6. Crear `.env.local` (URL + Anon Key de Supabase local)
- [ ] 7. Crear carpeta `src/utils/supabase/`
- [ ] 8. Crear `src/utils/supabase/client.js` (navegador)
- [ ] 9. Crear `src/utils/supabase/server.js` (servidor)
- [ ] 10. Crear store de Zustand
- [ ] 11. Probar que todo funciona

---

## PASO 1 — Encender Docker Desktop

### Antes de tocar la terminal:
1. Abre **Docker Desktop** (el icono de la ballena azul 🐳)
2. Espera a que ponga **"Engine running"** (motor funcionando)
3. Si no lo tienes instalado: https://www.docker.com/products/docker-desktop

```
┌─────────────────────────────┐
│  Docker Desktop             │
│  Status: 🟢 Engine running  │
│  ← Tiene que poner esto     │
└─────────────────────────────┘
```

> ⚠️ **Si Docker no está encendido**, `npx supabase start` dará error.

---

## PASO 2 — Crear proyecto Next.js

### Comando en terminal:
```bash
npx create-next-app@latest mi-proyecto
```

### Opciones que DEBES elegir (el profe dijo que sí a `src/`):
```
✔ Would you like to use TypeScript? … No / Yes (tú eliges, yo pongo No para el examen)
✔ Would you like to use ESLint? … No
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like your code inside a `src/` directory? … Yes  ← 🔥 IMPORTANTE: el profe dijo SÍ
✔ Would you like to use App Router? … Yes  ← 🔥 OBLIGATORIO
✔ Would you like to use Turbopack? … No
```

### Entrar al proyecto:
```bash
cd mi-proyecto
npm run dev
```
Abre `http://localhost:3000` para verificar que arranca.

---

## PASO 3 — Instalar dependencias

### Comando ÚNICO (copia y pega en terminal):
```bash
npm install @supabase/supabase-js @supabase/ssr zustand
```

### ¿Qué instala cada uno?
| Paquete | Para qué sirve |
|---------|----------------|
| `@supabase/supabase-js` | Cliente clásico de Supabase |
| `@supabase/ssr` | Cliente especial para Next.js (navegador + servidor) |
| `zustand` | Gestión de estado global (favoritos, carrito, etc.) |

---

## PASO 4 — Inicializar Supabase en el proyecto

### Comando (dentro de la carpeta del proyecto):
```bash
npx supabase init
```

### ¿Qué hace?
Crea una carpeta `supabase/` con la configuración inicial:
```
supabase/
  config.toml      ← configuración del proyecto local
  seed.sql         ← datos iniciales (lo rellenaremos después)
  migrations/      ← aquí irán los archivos SQL con los cambios de la DB
```

> 💡 **Solo se hace una vez** por proyecto. Si ya existe la carpeta `supabase/`, no hace falta repetirlo.

---

## PASO 5 — Arrancar Supabase en local

### Comando:
```bash
npx supabase start
```

### ¿Qué hace?
- Descarga las imágenes Docker la **primera vez** (tarda varios minutos, paciencia)
- Levanta todos los servicios: PostgreSQL, Auth, Storage, Studio...

### Al terminar, verás esto en la terminal:
```
Started supabase local development setup.

         API URL: http://localhost:54321
     GraphQL URL: http://localhost:54321/graphql/v1
  S3 Storage URL: http://localhost:54321/storage/v1/s3
          DB URL: postgresql://postgres:postgres@localhost:54322/postgres
      Studio URL: http://localhost:54323
    Inbucket URL: http://localhost:54324
      JWT secret: super-secret-jwt-token-with-at-least-32-characters-long
        anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 📌 PUERTOS IMPORTANTES (apunta estos)
| Servicio | Puerto local | URL de acceso |
|----------|-------------|---------------|
| API (PostgREST) | 54321 | http://localhost:54321 |
| Base de datos | 54322 | postgresql://... |
| Studio (Dashboard) | 54323 | http://localhost:54323 |
| Auth | 54324 | — |

> 🔑 **Para el examen**: El puerto que te importa es el **54321** (la API) y el **54323** (el dashboard visual).

---

## PASO 5.1 — Si falla en Windows: puerto ocupado

En Windows es frecuente que falle con:
```
Error: failed to start docker container: ... port is already allocated
```

### Solución: cambiar los puertos en `supabase/config.toml`

Abre `supabase/config.toml` y busca las líneas `port =` de cada servicio:

```toml
# supabase/config.toml

[api]
port = 57321      # cambia 54321 → 57321 (o cualquier puerto libre)

[db]
port = 57322
shadow_port = 57320

[studio]
port = 57323

[inbucket]
port = 57324
```

Guarda el archivo y vuelve a ejecutar:
```bash
npx supabase start
```

> Si cambias el puerto de la API, recuerda usar el nuevo puerto en `.env.local` (ej: `http://localhost:57321`).

---

## PASO 6 — Crear `.env.local` (variables de entorno)

### 6.1 Dónde encontrar los valores
Los valores te los dio el comando `npx supabase start` en la terminal (Paso 5).

Copia estos dos:
- **API URL** → `NEXT_PUBLIC_SUPABASE_URL`
- **anon key** → `NEXT_PUBLIC_SUPABASE_ANON_KEY`

```
┌─────────────────────────────────────────────────────┐
│  Terminal (salida de npx supabase start)            │
├─────────────────────────────────────────────────────┤
│  API URL: http://localhost:54321                    │
│  → Copiar esto: NEXT_PUBLIC_SUPABASE_URL            │
├─────────────────────────────────────────────────────┤
│  anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...  │
│  → Copiar esto: NEXT_PUBLIC_SUPABASE_ANON_KEY       │
└─────────────────────────────────────────────────────┘
```

### 6.2 Crear el archivo

En la **RAÍZ** del proyecto (al lado de `package.json`, fuera de `src/`):

```bash
touch .env.local
```

### 6.3 Contenido del archivo (copia y pega):
```env
# Supabase LOCAL (Docker)
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

> ⚠️ **REGLA**: Siempre `NEXT_PUBLIC_` al principio si la variable se usa en el navegador.
> El archivo `.env.local` NUNCA se sube a Git (ya viene en `.gitignore` por defecto).

---

## PASO 7 — Crear la carpeta de conexión

El profe quiere la carpeta `utils` dentro de `src/`.

### Comando en terminal:
```bash
mkdir -p src/utils/supabase
```

### Estructura que quedará:
```
mi-proyecto/
├── src/
│   └── utils/
│       └── supabase/
│           ├── client.js   ← navegador (botones, formularios)
│           └── server.js   ← servidor (Server Actions, API routes)
```

---

## PASO 8 — Crear `src/utils/supabase/client.js`

Este archivo es para **componentes del navegador** (botones de login, formularios, useEffect...).

### Código exacto (copia y pega):
```javascript
import { createClient } from '@supabase/supabase-js'
 

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY
export const supabase = createClient(supabaseUrl, supabaseKey)
```

### ¿Cómo se usa en un componente?
```javascript
'use client'
import { createClient } from '@/utils/supabase/client'

const supabase = createClient()
// Ya puedes usar: supabase.from('cursos').select('*')
```

---

## PASO 9 — Crear `src/utils/supabase/server.js`

Este archivo es para **Server Components** y **Server Actions** (lo que corre en el servidor, no en el navegador).

### Código exacto (copia y pega):
```javascript
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // Si estamos en un Server Component, no se pueden setear cookies
          }
        },
      },
    }
  )
}
```

### ¿Cómo se usa en un Server Component?
```javascript
import { createClient } from '@/utils/supabase/server'

export default async function MiPagina() {
  const supabase = await createClient()
  const { data } = await supabase.from('cursos').select('*')
  // data tiene los cursos
}
```

---

## PASO 10 — Crear store de Zustand

### 10.1 Crear carpeta:
```bash
mkdir -p src/stores
```

### 10.2 Crear `src/stores/favoritosStore.js`:
```javascript
import { create } from 'zustand'

export const useFavoritosStore = create((set) => ({
  favoritos: [],

  addFavorito: (curso) =>
    set((state) => ({
      favoritos: [...state.favoritos, curso],
    })),

  removeFavorito: (id) =>
    set((state) => ({
      favoritos: state.favoritos.filter((f) => f.id !== id),
    })),
}))
```

### 10.3 ¿Cómo se usa en un componente?
```javascript
'use client'
import { useFavoritosStore } from '@/stores/favoritosStore'

export default function BotonFavorito({ curso }) {
  const addFavorito = useFavoritosStore((state) => state.addFavorito)
  const removeFavorito = useFavoritosStore((state) => state.removeFavorito)
  const isFav = useFavoritosStore((state) =>
    state.favoritos.some((f) => f.id === curso.id)
  )

  return (
    <button onClick={() => isFav ? removeFavorito(curso.id) : addFavorito(curso)}>
      {isFav ? '❤️' : '🤍'}
    </button>
  )
}
```

---

## PASO 11 — Probar que todo funciona

### 11.1 Abrir Supabase Studio
Abre en tu navegador: **http://localhost:54323**

Desde aquí puedes:
- Crear tablas (Table Editor)
- Ejecutar SQL (SQL Editor)
- Ver usuarios de Auth
- Gestionar Storage

### 11.2 Crear la tabla `cursos` en Studio (para probar)
1. Ve a **Table Editor** → **New table**
2. Nombre: `cursos`
3. Columnas:
   - `id` → tipo `int8` → Default `nextval(...)` → Primary key ✅
   - `nombre` → tipo `text`
   - `modalidad` → tipo `text`
   - `horas` → tipo `int4`
   - `created_at` → tipo `timestamptz` → Default `now()`
4. Guarda (Save)

### 11.3 Crear página de test
Crea `src/app/test/page.js`:
```javascript
'use client'
import { useEffect } from 'react'
import { createClient } from '@/utils/supabase/client'

export default function TestPage() {
  useEffect(() => {
    async function probar() {
      const supabase = createClient()
      const { data, error } = await supabase.from('cursos').select('*')
      console.log('✅ Datos:', data)
      console.log('❌ Error:', error)
    }
    probar()
  }, [])

  return <h1>Abre la consola del navegador (F12) y mira si hay datos</h1>
}
```

### 11.4 Interpretar resultados
| Resultado en consola | Significado | Solución |
|----------------------|-------------|----------|
| `Datos: []` | Conexión OK, tabla vacía | Inserta datos en Supabase Studio |
| `Datos: [{...}, {...}]` | ✅ TODO PERFECTO | ¡A programar! |
| `Error: 401/403` | RLS bloquea el acceso | Ve al paso 12 |
| `Error: relation "cursos" does not exist` | Tabla no creada | Crea la tabla en Supabase Studio |

---

## PASO 12 — Si falla: desactivar RLS (lo que no viste en clase)

Si ves error 401/403 o el array vacío cuando hay datos, es porque **RLS está activado**.

### Solución rápida para el examen:
1. Abre **Supabase Studio** → **Table Editor** → `cursos`
2. Arriba a la derecha verás un botón verde que dice **"RLS enabled"**
3. **Haz clic y desactívalo** (pasa a gris)
4. Vuelve a probar tu página

```
┌────────────────────────────────────────────┐
│  Supabase Studio → Table Editor → cursos   │
├────────────────────────────────────────────┤
│  [RLS enabled]  ← Haz clic aquí            │
│       ↓                                    │
│  [RLS disabled] ← Debe quedar así (gris)   │
└────────────────────────────────────────────┘
```

> 💡 **Para el examen**: Desactivar RLS es válido si el profe no te pide explícitamente políticas de seguridad. Si te las pide, usa el apunte de RLS.

---

## 📁 ESTRUCTURA FINAL DEL PROYECTO

```
mi-proyecto/
├── .env.local                    ← Credenciales (NO subir a Git)
├── supabase/
│   ├── config.toml               ← Configuración de puertos Docker
│   ├── seed.sql
│   └── migrations/
├── src/
│   ├── app/
│   │   ├── cursos/
│   │   │   └── page.js           ← Tarea 1: CRUD cursos
│   │   ├── test/
│   │   │   └── page.js           ← Página para probar conexión
│   │   ├── layout.js
│   │   └── page.js
│   ├── utils/
│   │   └── supabase/
│   │       ├── client.js         ← Navegador (createBrowserClient)
│   │       └── server.js         ← Servidor (createServerClient)
│   └── stores/
│       └── favoritosStore.js     ← Zustand
├── document/                     ← 📂 TUS APUNTES (crea esta carpeta)
│   ├── 00-instalacion.md         ← Este archivo
│   ├── 01-crud-cursos.md
│   ├── 02-trigger-updated_at.md
│   └── 03-zustand-favoritos.md
└── package.json
```

---

## 🚀 COMANDOS RÁPIDOS (para copiar en el examen)

```bash
# 1. Crear proyecto
npx create-next-app@latest mi-proyecto

# 2. Instalar todo
npm install @supabase/supabase-js @supabase/ssr zustand

# 3. Crear carpetas
mkdir -p src/utils/supabase
mkdir -p src/stores
mkdir -p document

# 4. Inicializar Supabase (una sola vez)
npx supabase init

# 5. Arrancar Supabase local (Docker debe estar encendido)
npx supabase start

# 6. Ver estado y URLs
npx supabase status

# 7. Parar Supabase (conserva los datos)
npx supabase stop

# 8. Resetear la base de datos (borra todo y vuelve a empezar)
npx supabase db reset

# 9. Arrancar Next.js
npm run dev
```

---

## ⚠️ ERRORES COMUNES EN EL EXAMEN

| Error | Causa | Solución |
|-------|-------|----------|
| `Cannot connect to Docker` | Docker Desktop apagado | Enciende Docker Desktop antes de `npx supabase start` |
| `port is already allocated` | Puerto ocupado en Windows | Cambia puertos en `supabase/config.toml` |
| `useState` no funciona | Olvidaste `'use client'` | Pon `'use client'` arriba del todo |
| `process.env` es undefined | Olvidaste `NEXT_PUBLIC_` | El prefijo es obligatorio para el cliente |
| `Cannot find module '@/utils/...'` | No existe el archivo | Crea la carpeta y el archivo exacto |
| RLS 401/403 | Seguridad activada | Desactiva RLS en Supabase Studio |
| `cookies is not a function` | Versión antigua de Next.js | Usa `await cookies()` (Next.js 15+) |

---

## ✅ RESUMEN PARA EL EXAMEN

1. **Encender Docker Desktop** 🐳
2. `npx create-next-app@latest` → decir **SÍ** a `src/` y **SÍ** a App Router
3. `npm install @supabase/supabase-js @supabase/ssr zustand`
4. `npx supabase init` (una vez)
5. `npx supabase start` (esperar que descargue Docker la primera vez)
6. Copiar **API URL** y **anon key** de la terminal → `.env.local`
7. Crear `src/utils/supabase/client.js` y `server.js`
8. Crear `src/stores/favoritosStore.js`
9. Abrir **http://localhost:54323** para crear tablas
10. Si falla el CRUD → revisar RLS en Supabase Studio

**Siguiente apunte recomendado**: `01-crud-cursos.md`

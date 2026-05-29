# 📄 APUNTES RECICLABLES — RLS (Row Level Security)
## Seguridad en PostgreSQL/Supabase | Examen Práctico

> 🎯 **Objetivo**: Tener TODAS las respuestas listas para copiar y pegar.
> **Filosofía**: 80% del código SQL es SIEMPRE IGUAL. Solo cambias nombres de tabla/columna.
> **En el examen**: Abre esto, copia el bloque, busca y reemplaza, ejecuta.

---

## ✅ CHECKLIST DEL EXAMEN (Tarea de RLS)

- [ ] 1. Activar RLS en la tabla: `ALTER TABLE ... ENABLE ROW LEVEL SECURITY`
- [ ] 2. (Opcional) Crear función helper `mi_rol()` con `SECURITY DEFINER` + `STABLE`
- [ ] 3. Crear políticas `CREATE POLICY` con `USING` y/o `WITH CHECK`
- [ ] 4. Demostrar que funciona: simular usuario con `SET request.jwt.claims` y hacer SELECT/UPDATE

---

## 🧩 REGLA DE ORO: ¿QUÉ CAMBIA Y QUÉ ES FIJO?

| Parte | ¿Cambia? | Ejemplo fijo |
|-------|----------|--------------|
| Nombre de la tabla | ✅ SIEMPRE | `pedidos`, `cursos`, `notas`... |
| Nombre de la columna de usuario | ⚠️ Casi nunca | Siempre `usuario_id`, `cliente_id`, `autor_id`... |
| `ENABLE ROW LEVEL SECURITY` | ❌ NO | Siempre igual |
| `CREATE POLICY ... ON ... FOR ...` | ❌ NO | Siempre igual estructura |
| `auth.uid()` | ❌ NO | Siempre así |
| `auth.role()` | ❌ NO | Siempre así |
| `USING (...)` | ⚠️ A veces | Condición sobre filas existentes |
| `WITH CHECK (...)` | ⚠️ A veces | Condición sobre datos nuevos |
| `SECURITY DEFINER` | ❌ NO (si pide helper) | Siempre igual |
| `STABLE` | ❌ NO (si pide helper) | Siempre igual |
| `EXISTS (...)` | ⚠️ A veces | Subquery para tablas relacionadas |
| `FOR ALL` | ⚠️ A veces | Cuando el rol puede hacer todo |

---

## 🚀 PARTE 1: ACTIVAR RLS (Siempre igual)

```sql
-- ═══════════════════════════════════════════════════════════
-- PASO 0: Activar RLS en la tabla (SIEMPRE IGUAL)
-- ═══════════════════════════════════════════════════════════
-- CAMBIA AQUÍ: 'pedidos' → nombre de tu tabla
ALTER TABLE public.pedidos ENABLE ROW LEVEL SECURITY;

-- Si tienes varias tablas, repite:
-- ALTER TABLE public.cursos ENABLE ROW LEVEL SECURITY;
-- ALTER TABLE public.productos ENABLE ROW LEVEL SECURITY;
```

**¿Qué cambia?**
- Nombre de la tabla (`public.pedidos` → `public.cursos`, `public.notas`...)

**¿Qué NUNCA cambia?**
- `ALTER TABLE ... ENABLE ROW LEVEL SECURITY`
- Esquema `public.`

**IMPORTANTE**: Una tabla con RLS activado pero SIN políticas → **deniega TODO**. Devuelve 0 filas. No es error, es seguridad por defecto.

---

## 🚀 PARTE 2: FUNCIÓN HELPER `mi_rol()` (Si te piden roles de negocio)

**Cuándo usar**: Si el profe menciona roles como `'cliente'`, `'admin'`, `'staff'`, `'alumno'`... guardados en una tabla `perfiles`.

```sql
-- ═══════════════════════════════════════════════════════════
-- PASO 1 (opcional): Crear función helper mi_rol()
-- ═══════════════════════════════════════════════════════════
-- ESTO NUNCA CAMBIA. Es 100% estándar.
CREATE OR REPLACE FUNCTION public.mi_rol()
RETURNS text
LANGUAGE sql
STABLE
SECURITY DEFINER
AS $$
  SELECT rol FROM public.perfiles WHERE id = auth.uid()
$$;
```

**¿Qué cambia?**
- Nombre de la tabla de perfiles (`public.perfiles` → podría ser `public.usuarios`... pero casi siempre es `perfiles`)
- Nombre de la columna de rol (`rol` → casi siempre es `rol`)

**¿Qué NUNCA cambia?**
- `CREATE OR REPLACE FUNCTION public.mi_rol()`
- `RETURNS text`
- `LANGUAGE sql`
- `STABLE`
- `SECURITY DEFINER` ← OBLIGATORIO para evitar recursión infinita
- `SELECT rol FROM public.perfiles WHERE id = auth.uid()`

**¿Por qué `SECURITY DEFINER`?**
Sin él, la función haría `SELECT FROM perfiles` → PostgreSQL evalúa política de `perfiles` → la política llama a `mi_rol()` → bucle infinito. Con `SECURITY DEFINER`, la función corre como superadmin y salta las políticas.

**¿Por qué `STABLE`?**
PostgreSQL cachea el resultado dentro de la misma transacción. Si la tabla tiene 1000 filas, sin `STABLE` haría 1000 consultas a `perfiles`. Con `STABLE`, hace 1 sola.

---

## 🚀 PARTE 3: POLÍTICAS RECICLABLES

### Template base (cambia solo tabla, condición y nombre descriptivo)

```sql
-- ═══════════════════════════════════════════════════════════
-- PASO 2: Crear políticas
-- ═══════════════════════════════════════════════════════════

-- POLÍTICA 1: Un usuario solo ve sus propios registros (SELECT)
-- CAMBIA: nombre de tabla, nombre de columna de usuario, nombre de política
CREATE POLICY "usuario: ver sus registros"
  ON public.pedidos FOR SELECT
  USING ( cliente_id = auth.uid() );

-- POLÍTICA 2: Un usuario solo puede insertar registros como él mismo (INSERT)
CREATE POLICY "usuario: crear sus registros"
  ON public.pedidos FOR INSERT
  WITH CHECK ( cliente_id = auth.uid() );

-- POLÍTICA 3: Un usuario solo puede actualizar sus propios registros (UPDATE)
-- USING = qué filas puede tocar
-- WITH CHECK = cómo puede quedar la fila después
CREATE POLICY "usuario: editar sus registros"
  ON public.pedidos FOR UPDATE
  USING ( cliente_id = auth.uid() )
  WITH CHECK ( cliente_id = auth.uid() );

-- POLÍTICA 4: Un usuario solo puede borrar sus propios registros (DELETE)
CREATE POLICY "usuario: borrar sus registros"
  ON public.pedidos FOR DELETE
  USING ( cliente_id = auth.uid() );
```

**¿Qué cambia?**
- Nombre de la tabla (`public.pedidos`)
- Nombre de la columna de usuario (`cliente_id` → `usuario_id`, `autor_id`, `alumno_id`...)
- Nombre descriptivo de la política (texto entre comillas)
- La condición (`= auth.uid()`, `= 'admin'`, `EXISTS (...)`...)

**¿Qué NUNCA cambia?**
- `CREATE POLICY "..." ON public.x FOR ...`
- `USING (...)` para SELECT/UPDATE/DELETE
- `WITH CHECK (...)` para INSERT/UPDATE
- `auth.uid()` para identificar al usuario actual

---

## 🔥 VARIANTES QUE PUEDE PEDIR EL PROFE

### Variante A: Política pública (sin login)

**Cuándo usar**: Si el profe pide "que cualquiera pueda ver los productos/cursos publicados".

```sql
-- Cualquier usuario (incluso sin login = anon) puede ver filas públicas
CREATE POLICY "publico: ver publicados"
  ON public.cursos FOR SELECT
  USING (
    publicado = true
    -- o: estado = 'publicado'
    -- o: auth.role() = 'anon' OR publicado = true
  );
```

---

### Variante B: Rol admin puede hacer TODO (FOR ALL)

**Cuándo usar**: Si el profe pide "que el admin tenga acceso total".

```sql
-- Admin puede hacer SELECT, INSERT, UPDATE, DELETE sin restricciones
CREATE POLICY "admin: acceso total"
  ON public.pedidos FOR ALL
  USING ( public.mi_rol() = 'admin' )
  WITH CHECK ( public.mi_rol() = 'admin' );
```

**Diferencias**:
- `FOR ALL` en vez de `FOR SELECT/INSERT/UPDATE/DELETE`
- Requiere función `mi_rol()` (Variante helper)
- Necesita `USING` y `WITH CHECK` (ambos)

---

### Variante C: EXISTS (tablas relacionadas)

**Cuándo usar**: Si el profe pide "que un usuario vea solo los ítems de SUS pedidos" (la columna `cliente_id` está en `pedidos`, no en `pedido_items`).

```sql
-- El cliente ve los ítems de SUS pedidos
CREATE POLICY "cliente: ver items de sus pedidos"
  ON public.pedido_items FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM public.pedidos
      WHERE pedidos.id = pedido_items.pedido_id
        AND pedidos.cliente_id = auth.uid()
    )
  );
```

**Conceptos clave**:
- `EXISTS (SELECT 1 FROM ... WHERE ...)` → true si hay al menos una fila
- `SELECT 1` → más eficiente que `SELECT *`
- Une la tabla hija (`pedido_items`) con la padre (`pedidos`) por la clave foránea

---

### Variante D: WITH CHECK para validar datos nuevos

**Cuándo usar**: Si el profe pide "que un usuario no pueda cambiar el autor_id al editar" o "que solo pueda poner estado cancelada".

```sql
-- UPDATE: el usuario puede editar sus notas, pero NO puede cambiar el autor_id
CREATE POLICY "autor: editar sus notas"
  ON public.notas FOR UPDATE
  USING ( autor_id = auth.uid() )           -- qué filas puede tocar
  WITH CHECK ( autor_id = auth.uid() );       -- cómo puede quedar (autor no cambia)

-- UPDATE: solo puede cancelar reservas pendientes
CREATE POLICY "cliente: cancelar reserva"
  ON public.reservas FOR UPDATE
  USING (
    cliente_id = auth.uid()
    AND estado = 'pendiente'                -- solo puede tocar las pendientes
  )
  WITH CHECK (
    cliente_id = auth.uid()
    AND estado = 'cancelada'                -- solo puede dejarlas como cancelada
  );
```

**Regla mnemotécnica**:
- `USING` = **filtro** sobre filas existentes (qué puedo tocar)
- `WITH CHECK` = **validación** de los datos nuevos (cómo puede quedar)

---

### Variante E: Combinar múltiples políticas (OR lógico)

**Cuándo usar**: Si el profe pide "que el autor vea todo y los demás solo lo público".

```sql
-- Política 1: el autor ve TODOS sus documentos (borradores y publicados)
CREATE POLICY "autor: ver sus documentos"
  ON public.documentos FOR SELECT
  USING ( autor_id = auth.uid() );

-- Política 2: cualquier autenticado ve los documentos NO borrador
CREATE POLICY "autenticado: ver publicados"
  ON public.documentos FOR SELECT
  USING (
    auth.role() = 'authenticated'
    AND borrador = false
  );

-- Resultado: el autor ve TODO (pasa la política 1)
-- Los demás ven solo los publicados (pasan la política 2)
-- Anónimos ven 0 filas (no pasan ninguna)
```

**IMPORTANTE**: Las políticas se combinan con **OR**. Una fila aparece si pasa AL MENOS UNA política.

---

## 🧪 PARTE 4: DEMOSTRAR QUE FUNCIONA (Pruebas en SQL Editor)

### Simular un usuario y probar

```sql
-- ═══════════════════════════════════════════════════════════
-- PASO 3: Demostrar que funciona
-- ═══════════════════════════════════════════════════════════

-- 1. Simular ser un usuario autenticado específico
SET request.jwt.claims = '{"sub":"UUID-DEL-USUARIO","role":"authenticated"}';

-- 2. Probar SELECT (debe devolver SOLO sus filas)
SELECT * FROM public.pedidos;

-- 3. Probar INSERT (debe funcionar solo si cliente_id = su UUID)
INSERT INTO public.pedidos (cliente_id, producto, precio)
VALUES ('UUID-DEL-USUARIO', 'Producto A', 10.00);

-- 4. Probar UPDATE (debe funcionar solo en sus filas)
UPDATE public.pedidos SET precio = 20.00 WHERE id = 1;

-- 5. Probar DELETE (debe funcionar solo en sus filas)
DELETE FROM public.pedidos WHERE id = 1;

-- 6. Simular ser otro usuario (para probar que NO ve las filas del primero)
SET request.jwt.claims = '{"sub":"OTRO-UUID","role":"authenticated"}';
SELECT * FROM public.pedidos;  -- Debe devolver 0 filas o solo las suyas

-- 7. Simular usuario anónimo
SET request.jwt.claims = '{"role":"anon"}';
SELECT * FROM public.pedidos;  -- Debe devolver 0 filas (si no hay política pública)
```

**¿Qué cambia?**
- El UUID en `sub`
- Nombre de la tabla
- Campos del INSERT/UPDATE

**¿Qué NUNCA cambia?**
- `SET request.jwt.claims = '{"sub":"...","role":"authenticated"}'`
- Estructura de las queries de prueba

---

## 🎯 MAPA DE PROBABILIDAD: ¿QUÉ PUEDE CAER?

| Tema | Probabilidad | Qué estudiar |
|------|-------------|--------------|
| **Activar RLS + política básica** (`auth.uid() = cliente_id`) | **90%** | Template base completo |
| **Política pública** (`publicado = true`) | **60%** | Variante A |
| **FOR ALL para admin** | **50%** | Variante B + función helper |
| **EXISTS para tablas relacionadas** | **40%** | Variante C |
| **WITH CHECK para validar datos** | **35%** | Variante D |
| **Múltiples políticas combinadas** | **30%** | Variante E |
| **Simular usuario con SET request.jwt.claims** | **85%** | Parte 4 completa |
| **Función helper mi_rol()** | **45%** | Parte 2 completa |

---

## ⚡ CHEAT SHEET ULTRA-RÁPIDO (Copiar y pegar en el examen)

### Activar RLS
```sql
ALTER TABLE public.TABLA ENABLE ROW LEVEL SECURITY;
```

### Función helper (si pide roles)
```sql
CREATE OR REPLACE FUNCTION public.mi_rol() RETURNS text LANGUAGE sql STABLE SECURITY DEFINER AS $$ SELECT rol FROM public.perfiles WHERE id = auth.uid() $$;
```

### Política básica (ver solo lo mío)
```sql
CREATE POLICY "ver_mios" ON public.TABLA FOR SELECT USING ( usuario_id = auth.uid() );
CREATE POLICY "insertar_mio" ON public.TABLA FOR INSERT WITH CHECK ( usuario_id = auth.uid() );
CREATE POLICY "editar_mio" ON public.TABLA FOR UPDATE USING ( usuario_id = auth.uid() ) WITH CHECK ( usuario_id = auth.uid() );
CREATE POLICY "borrar_mio" ON public.TABLA FOR DELETE USING ( usuario_id = auth.uid() );
```

### Admin acceso total
```sql
CREATE POLICY "admin_total" ON public.TABLA FOR ALL USING ( public.mi_rol() = 'admin' ) WITH CHECK ( public.mi_rol() = 'admin' );
```

### EXISTS (ítems de mis pedidos)
```sql
CREATE POLICY "ver_items" ON public.ITEMS FOR SELECT USING ( EXISTS ( SELECT 1 FROM public.PEDIDOS WHERE PEDIDOS.id = ITEMS.pedido_id AND PEDIDOS.cliente_id = auth.uid() ) );
```

### Probar
```sql
SET request.jwt.claims = '{"sub":"UUID","role":"authenticated"}';
SELECT * FROM public.TABLA;
```

---

## 🐛 ERRORES COMUNES Y SOLUCIONES RÁPIDAS

| Error | Por qué pasa | Solución en 5 segundos |
|-------|-------------|------------------------|
| `SELECT devuelve 0 filas` | RLS activado pero sin políticas | Crea al menos una política `FOR SELECT` |
| `INSERT no funciona` | No hay política `FOR INSERT` | Añade `CREATE POLICY ... FOR INSERT WITH CHECK (...)` |
| `UPDATE no funciona` | No hay política `FOR UPDATE` | Añade política con `USING` y `WITH CHECK` |
| `DELETE no borra nada` | No hay política `FOR DELETE` | Añade `CREATE POLICY ... FOR DELETE USING (...)` |
| `mi_rol() devuelve null` | El usuario no tiene perfil | Inserta fila en `perfiles` con ese `id` y `rol` |
| `mi_rol() da error de recursión` | Falta `SECURITY DEFINER` | Añade `SECURITY DEFINER` a la función |
| `service_role no ve nada` | No, `service_role` bypasea RLS | Si `service_role` falla, el problema es otro (tabla no existe, etc.) |
| `auth.uid() es null` | Petición sin token / simulación mal | Revisa `SET request.jwt.claims` con `"sub":"UUID"` |
| `violates row-level security policy` | WITH CHECK falla | El dato nuevo no cumple la condición de `WITH CHECK` |
| `permission denied for table auth.users` | Falta `SECURITY DEFINER` en función que lee `auth.users` | Añade `SECURITY DEFINER` |

---

## ✅ CHECKLIST FINAL ANTES DE ENTREGAR (RLS)

- [ ] `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` ejecutado
- [ ] Al menos una política `FOR SELECT` creada (si no, devuelve 0 filas)
- [ ] Política `FOR INSERT` con `WITH CHECK` si deben crear registros
- [ ] Política `FOR UPDATE` con `USING` + `WITH CHECK` si deben editar
- [ ] Política `FOR DELETE` con `USING` si deben borrar
- [ ] Si uso `mi_rol()`, tiene `SECURITY DEFINER` y `STABLE`
- [ ] Si uso `EXISTS`, la subquery tiene `SELECT 1` (no `SELECT *`)
- [ ] Demostré que funciona con `SET request.jwt.claims` + SELECT/INSERT/UPDATE
- [ ] Probé con un UUID que NO tiene permiso → debe devolver 0 filas
- [ ] Probé con `service_role` → debe ver todo (bypass RLS)

---

## 📝 EJEMPLOS DE PREGUNTAS TIPO EXAMEN (con respuestas listas)

### Pregunta 1: "Activa RLS en la tabla cursos y haz que cada usuario solo vea sus cursos"

**Respuesta completa lista para copiar:**

```sql
-- Activar RLS
ALTER TABLE public.cursos ENABLE ROW LEVEL SECURITY;

-- Política: solo ve los cursos donde usuario_id es el suyo
CREATE POLICY "usuario: ver sus cursos"
  ON public.cursos FOR SELECT
  USING ( usuario_id = auth.uid() );

-- Demostración
SET request.jwt.claims = '{"sub":"11111111-1111-1111-1111-111111111111","role":"authenticated"}';
SELECT * FROM public.cursos;  -- Solo devuelve los cursos de ese usuario
```

---

### Pregunta 2: "Crea un sistema donde los alumnos vean solo sus matrículas, pero el admin vea todo"

**Respuesta completa lista para copiar:**

```sql
-- Activar RLS
ALTER TABLE public.matriculas ENABLE ROW LEVEL SECURITY;

-- Función helper (si no existe)
CREATE OR REPLACE FUNCTION public.mi_rol()
RETURNS text LANGUAGE sql STABLE SECURITY DEFINER
AS $$ SELECT rol FROM public.perfiles WHERE id = auth.uid() $$;

-- Política 1: alumno ve solo sus matrículas
CREATE POLICY "alumno: ver sus matriculas"
  ON public.matriculas FOR SELECT
  USING ( alumno_id = auth.uid() );

-- Política 2: admin ve todo
CREATE POLICY "admin: ver todas las matriculas"
  ON public.matriculas FOR SELECT
  USING ( public.mi_rol() = 'admin' );

-- Demostración alumno
SET request.jwt.claims = '{"sub":"ALUMNO-UUID","role":"authenticated"}';
SELECT * FROM public.matriculas;  -- Solo las suyas

-- Demostración admin
SET request.jwt.claims = '{"sub":"ADMIN-UUID","role":"authenticated"}';
SELECT * FROM public.matriculas;  -- Todas
```

---

### Pregunta 3: "Un cliente puede ver los items de sus pedidos, pero no los de otros"

**Respuesta completa lista para copiar:**

```sql
-- Activar RLS en ambas tablas
ALTER TABLE public.pedidos ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.pedido_items ENABLE ROW LEVEL SECURITY;

-- Política en pedidos: cliente ve solo sus pedidos
CREATE POLICY "cliente: ver sus pedidos"
  ON public.pedidos FOR SELECT
  USING ( cliente_id = auth.uid() );

-- Política en pedido_items: cliente ve items de sus pedidos (EXISTS)
CREATE POLICY "cliente: ver items de sus pedidos"
  ON public.pedido_items FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM public.pedidos
      WHERE pedidos.id = pedido_items.pedido_id
        AND pedidos.cliente_id = auth.uid()
    )
  );

-- Demostración
SET request.jwt.claims = '{"sub":"CLIENTE-UUID","role":"authenticated"}';
SELECT * FROM public.pedido_items;  -- Solo items de sus pedidos
```

---

### Pregunta 4: "Un autor puede editar sus notas, pero no puede cambiar el autor_id"

**Respuesta completa lista para copiar:**

```sql
ALTER TABLE public.notas ENABLE ROW LEVEL SECURITY;

-- Ver sus notas
CREATE POLICY "autor: ver sus notas"
  ON public.notas FOR SELECT
  USING ( autor_id = auth.uid() );

-- Editar sus notas, pero autor_id no puede cambiar
CREATE POLICY "autor: editar sus notas"
  ON public.notas FOR UPDATE
  USING ( autor_id = auth.uid() )
  WITH CHECK ( autor_id = auth.uid() );

-- Demostración: intentar cambiar autor_id (debe fallar)
SET request.jwt.claims = '{"sub":"AUTOR-UUID","role":"authenticated"}';
UPDATE public.notas SET autor_id = 'OTRO-UUID' WHERE id = 1;
-- ERROR: violates row-level security policy
```

---

### Pregunta 5: "Crea una política pública para que cualquiera vea los cursos publicados, pero solo el autor edite los suyos"

**Respuesta completa lista para copiar:**

```sql
ALTER TABLE public.cursos ENABLE ROW LEVEL SECURITY;

-- Pública: cualquiera (anon o autenticado) ve los publicados
CREATE POLICY "publico: ver cursos publicados"
  ON public.cursos FOR SELECT
  USING ( publicado = true );

-- Autor: ve TODOS sus cursos (publicados o no)
CREATE POLICY "autor: ver sus cursos"
  ON public.cursos FOR SELECT
  USING ( autor_id = auth.uid() );

-- Autor: edita solo los suyos
CREATE POLICY "autor: editar sus cursos"
  ON public.cursos FOR UPDATE
  USING ( autor_id = auth.uid() )
  WITH CHECK ( autor_id = auth.uid() );

-- Demostración anónimo
SET request.jwt.claims = '{"role":"anon"}';
SELECT * FROM public.cursos;  -- Solo los publicados

-- Demostración autor
SET request.jwt.claims = '{"sub":"AUTOR-UUID","role":"authenticated"}';
SELECT * FROM public.cursos;  -- Todos sus cursos (pasa la política 2)
```

---

> 💡 **Tip final**: En el examen, lee bien si pide `USING`, `WITH CHECK` o ambos. Si pide "que un usuario pueda ver", usa `USING`. Si pide "que un usuario pueda crear/editar", usa `WITH CHECK` (y `USING` también para UPDATE/DELETE).

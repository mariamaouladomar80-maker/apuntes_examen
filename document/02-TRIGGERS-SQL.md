# 📄 APUNTE 02 — TRIGGERS SQL ESTÁNDAR (Reciclable)
## Funciones + Triggers en PostgreSQL para cualquier tabla

> 🎯 **Objetivo**: Template SQL que sirve para cualquier tabla. Solo cambias el nombre.
> Guarda en: `tu-proyecto/document/02-triggers-sql.md`
> **En el examen**: Abre esto, copia el bloque SQL, cambia `'cursos'` por tu tabla, ejecuta.

---

## ✅ CHECKLIST DE LA TAREA 2 (para marcar en el examen)

- [ ] 1. Añadir columna `updated_at` (timestamptz) a la tabla
- [ ] 2. Crear función `set_updated_at()` en PL/pgSQL
- [ ] 3. Crear trigger `BEFORE UPDATE` sobre la tabla
- [ ] 4. Demostrar que funciona actualizando un registro

---

## 🧩 ¿QUÉ CAMBIA Y QUÉ ES FIJO?

| Parte | ¿Cambia? | Ejemplo |
|-------|----------|---------|
| Nombre de la tabla | ✅ SÍ | `'cursos'`, `'productos'`, `'empleados'`... |
| Nombre del campo fecha | ⚠️ Casi nunca | Siempre `updated_at` (así lo pide el profe) |
| Nombre de la función | ⚠️ Opcional | `set_updated_at()` (estándar) |
| Nombre del trigger | ⚠️ Opcional | `trigger_set_updated_at` (estándar) |
| El cuerpo de la función | ❌ NO | Siempre `NEW.updated_at = NOW(); RETURN NEW;` |
| `BEFORE UPDATE` | ❌ NO | Siempre igual |
| `FOR EACH ROW` | ❌ NO | Siempre igual |
| `RETURNS trigger` | ❌ NO | Siempre igual |
| `LANGUAGE plpgsql` | ❌ NO | Siempre igual |

---

## 🚀 TEMPLATE SQL COMPLETO (Copia y pega esto en SQL Editor)

```sql
-- ═══════════════════════════════════════════════════════════
-- PASO 1: Añadir columna updated_at (si no existe)
-- ═══════════════════════════════════════════════════════════
-- CAMBIA AQUÍ: 'cursos' → nombre de tu tabla
ALTER TABLE cursos ADD COLUMN IF NOT EXISTS updated_at timestamptz;

-- ═══════════════════════════════════════════════════════════
-- PASO 2: Crear la función set_updated_at()
-- ═══════════════════════════════════════════════════════════
-- ESTO NUNCA CAMBIA. Es 100% estándar.
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.updated_at = NOW();   -- asigna la fecha/hora actual
  RETURN NEW;               -- devuelve la fila modificada
END;
$$;

-- ═══════════════════════════════════════════════════════════
-- PASO 3: Crear el trigger BEFORE UPDATE
-- ═══════════════════════════════════════════════════════════
-- CAMBIA AQUÍ: 'cursos' → nombre de tu tabla
CREATE TRIGGER trigger_set_updated_at
  BEFORE UPDATE ON cursos     -- se dispara ANTES de cada UPDATE
  FOR EACH ROW                -- para cada fila modificada
  EXECUTE FUNCTION set_updated_at();
```

---

## ✏️ CÓMO ADAPTAR EN 10 SEGUNDOS

### Si el profe te pide otra tabla (ej: `productos`):

```sql
-- Solo cambia 'cursos' por 'productos' en 2 sitios:

-- PASO 1:
ALTER TABLE productos ADD COLUMN IF NOT EXISTS updated_at timestamptz;

-- PASO 2: (la función NO cambia, es universal)

-- PASO 3:
CREATE TRIGGER trigger_set_updated_at
  BEFORE UPDATE ON productos   -- ← aquí cambia
  FOR EACH ROW
  EXECUTE FUNCTION set_updated_at();
```

**Listo.** Eso es todo lo que cambia. La función `set_updated_at()` es la misma para todas las tablas.

---

## 🧪 PASO 4 — DEMOSTRAR QUE FUNCIONA (Queries de prueba)

Después de crear el trigger, ejecuta esto para demostrar al profe que funciona:

```sql
-- 1. Ver un registro ANTES de actualizar
SELECT id, nombre, updated_at FROM cursos LIMIT 1;
-- updated_at debería ser NULL (o la fecha anterior)

-- 2. Actualizar el nombre (o cualquier campo)
UPDATE cursos SET nombre = 'Nombre Modificado' WHERE id = 1;

-- 3. Ver el mismo registro DESPUÉS de actualizar
SELECT id, nombre, updated_at FROM cursos WHERE id = 1;
-- updated_at ahora debería tener la fecha/hora actual (NOW())
```

### ¿Qué debe pasar?
| Antes del UPDATE | Después del UPDATE |
|------------------|-------------------|
| `updated_at = NULL` | `updated_at = 2026-05-24 14:30:00+00` |
| o `updated_at = fecha_antigua` | `updated_at = fecha_actual` |

> ✅ Si `updated_at` cambia automáticamente → **el trigger funciona perfectamente.**

---

## 📸 CAPTURA DE PANTALLA EN TEXTO (para el examen)

Si el profe te pide "demostrar que funciona", en el SQL Editor de Supabase:

```
┌─────────────────────────────────────────────────────────────┐
│  SQL Editor → New Query                                     │
├─────────────────────────────────────────────────────────────┤
│  Query 1: SELECT id, nombre, updated_at FROM cursos;        │
│  Resultado:                                                 │
│  ┌────┬────────────────┬─────────────┐                    │
│  │ id │ nombre         │ updated_at  │                    │
│  ├────┼────────────────┼─────────────┤                    │
│  │  1 │ Curso Antiguo  │ NULL        │ ← antes          │
│  └────┴────────────────┴─────────────┘                    │
├─────────────────────────────────────────────────────────────┤
│  Query 2: UPDATE cursos SET nombre='Nuevo' WHERE id=1;    │
│  Resultado: UPDATE 1                                        │
├─────────────────────────────────────────────────────────────┤
│  Query 3: SELECT id, nombre, updated_at FROM cursos;      │
│  Resultado:                                                 │
│  ┌────┬────────────────┬─────────────────────────────┐    │
│  │ id │ nombre         │ updated_at                  │    │
│  ├────┼────────────────┼─────────────────────────────┤    │
│  │  1 │ Nuevo          │ 2026-05-24 14:30:00.123+00  │ ← ahora tiene fecha │
│  └────┴────────────────┴─────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔥 VARIANTES ÚTILES (por si el profe pide algo similar)

### Variante A: Trigger para `created_at` (si no usas DEFAULT now())

```sql
-- Función para rellenar created_at al insertar
CREATE OR REPLACE FUNCTION set_created_at()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.created_at = NOW();
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_set_created_at
  BEFORE INSERT ON cursos
  FOR EACH ROW
  EXECUTE FUNCTION set_created_at();
```

### Variante B: Trigger para contador (actualizar un campo automático)

```sql
-- Cada vez que se actualiza, suma 1 a 'numero_modificaciones'
CREATE OR REPLACE FUNCTION incrementar_contador()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.numero_modificaciones = COALESCE(OLD.numero_modificaciones, 0) + 1;
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_contador
  BEFORE UPDATE ON cursos
  FOR EACH ROW
  EXECUTE FUNCTION incrementar_contador();
```

> 💡 **Tip**: `COALESCE(valor, 0)` significa "si es NULL, usa 0". Evita errores.

---

## 🐛 ERRORES COMUNES Y SOLUCIONES

| Error | Por qué pasa | Solución |
|-------|-------------|----------|
| `function does not exist` | La función no se creó antes del trigger | Crea la función (PASO 2) ANTES del trigger (PASO 3) |
| `relation "cursos" does not exist` | La tabla no existe | Crea la tabla primero (apunte 01, PASO 0) |
| `column "updated_at" does not exist` | No ejecutaste el ALTER TABLE | Ejecuta PASO 1 antes |
| `trigger already exists` | Lo creaste dos veces | Usa `CREATE OR REPLACE TRIGGER` o borra primero: `DROP TRIGGER ...` |
| `updated_at` no cambia | El trigger es AFTER en vez de BEFORE | Debe ser `BEFORE UPDATE`, no `AFTER UPDATE` |
| `updated_at` sigue NULL | El trigger no se disparó | Solo se dispara en UPDATE, no en INSERT. Haz un UPDATE de prueba. |

---

## ✅ RESUMEN PARA EL EXAMEN (Tarea 2)

1. **Abre** este archivo (`02-triggers-sql.md`)
2. **Copia** el bloque SQL completo (PASO 1 + 2 + 3)
3. **Pégalo** en Supabase Studio → SQL Editor
4. **Cambia** `'cursos'` por el nombre de tu tabla (2 sitios)
5. **Ejecuta** (`Run`)
6. **Demuestra** que funciona con las queries de prueba (PASO 4)
7. **Captura** el resultado (si el profe pide evidencia)

**La función `set_updated_at()` NUNCA cambia.** Es universal.
**Solo cambias el nombre de la tabla en 2 sitios.**

---

## 🚀 COMANDOS RÁPIDOS (para el examen)

```sql
-- 1. Añadir columna
ALTER TABLE cursos ADD COLUMN IF NOT EXISTS updated_at timestamptz;

-- 2. Crear función (SIEMPRE IGUAL)
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS trigger LANGUAGE plpgsql AS $$
BEGIN NEW.updated_at = NOW(); RETURN NEW; END;
$$;

-- 3. Crear trigger (solo cambia 'cursos')
CREATE TRIGGER trigger_set_updated_at
  BEFORE UPDATE ON cursos FOR EACH ROW
  EXECUTE FUNCTION set_updated_at();

-- 4. Probar
UPDATE cursos SET nombre = 'Prueba' WHERE id = 1;
SELECT id, nombre, updated_at FROM cursos WHERE id = 1;
```

**Siguiente apunte**: `03-rls-seguridad.md` (backup para el ordinario)

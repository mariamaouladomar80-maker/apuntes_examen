# 📄 APUNTES RECICLABLES — TRIGGERS Y FUNCIONES POSTGRESQL
## Examen Práctico | Basado en Parcial 1846 + Teoría Previa

> 🎯 **Objetivo**: Tener TODAS las variantes de triggers/funciones que el profe puede pedir.
> **Estrategia**: Copia el bloque, cambia el nombre de la tabla/columna, ejecuta.
> Guarda como: `apuntes-sql-triggers.md`

---

## ✅ CHECKLIST DEL EXAMEN (Tarea de Triggers)

- [ ] 1. Añadir columna `updated_at` (timestamptz) a la tabla
- [ ] 2. Crear función en PL/pgSQL (o SQL puro si es simple)
- [ ] 3. Crear trigger `BEFORE UPDATE` (o `BEFORE INSERT` / `AFTER INSERT`)
- [ ] 4. Demostrar que funciona: `SELECT` → `UPDATE` → `SELECT`

---

## 🧩 REGLA DE ORO: ¿QUÉ CAMBIA Y QUÉ ES FIJO?

| Parte | ¿Cambia? | Ejemplo fijo |
|-------|----------|--------------|
| Nombre de la tabla | ✅ SIEMPRE | `'cursos'`, `'productos'`, `'empleados'` |
| Nombre del campo fecha | ⚠️ Casi nunca | Siempre `updated_at` (así lo pide el profe) |
| Nombre de la función | ⚠️ Opcional | `set_updated_at()` (estándar) |
| Nombre del trigger | ⚠️ Opcional | `trigger_set_updated_at` (estándar) |
| Cuerpo de la función | ❌ NO | `NEW.updated_at = NOW(); RETURN NEW;` |
| `BEFORE UPDATE` | ❌ NO | Siempre igual |
| `FOR EACH ROW` | ❌ NO | Siempre igual |
| `RETURNS trigger` | ❌ NO | Siempre igual |
| `LANGUAGE plpgsql` | ❌ NO | Siempre igual |
| `CREATE OR REPLACE` | ❌ NO | Siempre usarlo (evita errores si ya existe) |

---

## 🚀 TEMPLATE BASE (El del Parcial 1846)

```sql
-- ═══════════════════════════════════════════════════════════
-- PASO 1: Añadir columna updated_at (si no existe)
-- ═══════════════════════════════════════════════════════════
-- CAMBIA AQUÍ: 'cursos' → nombre de tu tabla
ALTER TABLE cursos 
ADD COLUMN IF NOT EXISTS updated_at timestamptz;

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

**Solo cambias `'cursos'` en 2 sitios.** La función es universal.

---

## 🧪 PASO 4 — DEMOSTRAR QUE FUNCIONA

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
| `updated_at = NULL` | `updated_at = 2026-05-28 14:30:00+00` |
| o `updated_at = fecha_antigua` | `updated_at = fecha_actual` |

> ✅ Si `updated_at` cambia automáticamente → **el trigger funciona perfectamente.**

---

## 🔥 VARIANTES QUE PUEDE PEDIR EL PROFE

### Variante A: Trigger para `created_at` (si no usas DEFAULT now())

**Cuándo usar**: Si la tabla NO tiene `DEFAULT now()` en `created_at` y quieres rellenarlo automáticamente al insertar.

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
  BEFORE INSERT ON cursos    -- ← BEFORE INSERT, no UPDATE
  FOR EACH ROW
  EXECUTE FUNCTION set_created_at();
```

**Diferencias con el base**:
- Evento: `BEFORE INSERT` en vez de `BEFORE UPDATE`
- Campo: `created_at` en vez de `updated_at`

---

### Variante B: Trigger contador (número de modificaciones)

**Cuándo usar**: Si el profe pide "llevar un contador de cuántas veces se ha editado un registro".

```sql
-- Primero añadir la columna contador (si no existe)
ALTER TABLE cursos 
ADD COLUMN IF NOT EXISTS numero_modificaciones int DEFAULT 0;

-- Función contador
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

**Conceptos clave**:
- `OLD.numero_modificaciones` → valor ANTES del update
- `COALESCE(valor, 0)` → "si es NULL, usa 0" (evita errores)
- `NEW.numero_modificaciones` → valor DESPUÉS (el que se guardará)

---

### Variante C: Trigger AFTER INSERT para auditoría / log

**Cuándo usar**: Si el profe pide "registrar automáticamente en otra tabla cada vez que se inserta algo".

```sql
-- Crear tabla de logs primero
CREATE TABLE IF NOT EXISTS logs_cursos (
  id serial PRIMARY KEY,
  curso_id int NOT NULL,
  accion text NOT NULL DEFAULT 'INSERT',
  creado_en timestamptz NOT NULL DEFAULT NOW()
);

-- Función que inserta en la tabla log
CREATE OR REPLACE FUNCTION log_nuevo_curso()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO logs_cursos (curso_id, accion)
  VALUES (NEW.id, 'INSERT');
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_log_curso
  AFTER INSERT ON cursos     -- ← AFTER: el dato ya está guardado
  FOR EACH ROW
  EXECUTE FUNCTION log_nuevo_curso();
```

**Diferencias con el base**:
- `AFTER INSERT` en vez de `BEFORE UPDATE`
- Usa `NEW.id` para leer el ID ya insertado
- Hace `INSERT INTO otra_tabla` dentro de la función

---

### Variante D: Validación con RAISE EXCEPTION (impedir datos inválidos)

**Cuándo usar**: Si el profe pide "que no se puedan poner horas negativas" o "precio menor que 0".

```sql
-- Añadir columna si no existe (ejemplo: horas)
ALTER TABLE cursos 
ADD COLUMN IF NOT EXISTS horas int;

-- Función validadora
CREATE OR REPLACE FUNCTION validar_horas()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  IF NEW.horas < 0 THEN
    RAISE EXCEPTION 'Las horas no pueden ser negativas. Valor intentado: %', NEW.horas;
  END IF;

  IF NEW.horas > 500 THEN
    RAISE EXCEPTION 'Las horas no pueden superar 500. Valor intentado: %', NEW.horas;
  END IF;

  RETURN NEW;  -- si pasa la validación, permite la operación
END;
$$;

CREATE TRIGGER trigger_validar_horas
  BEFORE INSERT OR UPDATE ON cursos   -- ← ambos eventos
  FOR EACH ROW
  EXECUTE FUNCTION validar_horas();
```

**Conceptos clave**:
- `RAISE EXCEPTION 'mensaje %', variable` → cancela la operación y devuelve error
- `BEFORE INSERT OR UPDATE` → se dispara en ambos eventos
- `RETURN NEW` → si pasa la validación, deja continuar

---

### Variante E: Trigger BEFORE INSERT que calcula un campo automático

**Cuándo usar**: Si el profe pide "calcular el total automáticamente antes de guardar".

```sql
-- Ejemplo: tabla pedidos donde total = precio * cantidad
ALTER TABLE pedidos 
ADD COLUMN IF NOT EXISTS total numeric(8,2);

CREATE OR REPLACE FUNCTION calcular_total_pedido()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.total := NEW.precio_unitario * NEW.cantidad;
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_calcular_total
  BEFORE INSERT ON pedidos
  FOR EACH ROW
  EXECUTE FUNCTION calcular_total_pedido();
```

**Conceptos clave**:
- `NEW.campo := valor` → asigna valor calculado antes de insertar
- El usuario NO necesita enviar el campo `total`, se calcula solo

---

### Variante F: Función SQL simple (sin PL/pgSQL)

**Cuándo usar**: Si el profe pide una función simple de cálculo (precio con IVA, descuento...).

```sql
-- Función SQL puro (una sola consulta, sin lógica condicional)
CREATE OR REPLACE FUNCTION precio_con_iva(precio numeric)
RETURNS numeric
LANGUAGE sql
STABLE   -- resultado no cambia dentro de la misma transacción
AS $$
  SELECT precio * 1.21;
$$;

-- Usarla:
SELECT precio_con_iva(100.00);  -- 121.00
```

**Diferencias con PL/pgSQL**:
- `LANGUAGE sql` en vez de `LANGUAGE plpgsql`
- No usa `BEGIN/END`, solo una expresión
- No puede usar `IF`, bucles ni variables
- Más rápida para cálculos simples

---

### Variante G: Función con SECURITY DEFINER (permisos elevados)

**Cuándo usar**: Si la función necesita leer tablas internas como `auth.users` y el usuario normal no tiene permisos.

```sql
CREATE OR REPLACE FUNCTION obtener_email_usuario(user_id uuid)
RETURNS text
LANGUAGE sql
SECURITY DEFINER   -- ← se ejecuta con permisos del creador, no del llamador
STABLE
AS $$
  SELECT email FROM auth.users WHERE id = user_id;
$$;
```

**Conceptos clave**:
- `SECURITY DEFINER` → la función corre con permisos de quien la CREÓ (superadmin)
- Útil para leer `auth.users` desde políticas RLS
- **Riesgo**: si la función tiene errores, un usuario podría hacer cosas con privilegios totales
- Usar solo cuando sea estrictamente necesario

---

### Variante H: Trigger para crear perfil al registrarse (Flex/Supabase Auth)

**Cuándo usar**: Si el profe pide "que al crear un usuario en auth.users se cree automáticamente un perfil".

```sql
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS trigger
LANGUAGE plpgsql
SECURITY DEFINER   -- necesario para acceder a auth.users
AS $$
BEGIN
  INSERT INTO public.perfiles (id, nombre)
  VALUES (
    NEW.id,                                   -- UUID del nuevo usuario
    NEW.raw_user_meta_data->>'full_name'       -- extrae del JSON de metadata
  );
  RETURN NEW;
END;
$$;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT
  ON auth.users
  FOR EACH ROW
  EXECUTE FUNCTION public.handle_new_user();
```

**Conceptos clave**:
- Se dispara sobre `auth.users` (tabla interna de Supabase)
- `NEW.raw_user_meta_data->>'full_name'` → extrae campo de JSON
- `SECURITY DEFINER` obligatorio porque `auth.users` es protegida

---

## 📊 RESUMEN: ¿Qué trigger usar según lo que pida el profe?

| Si te pide... | Usa | Evento | Palabras clave |
|---------------|-----|--------|----------------|
| "Registrar última modificación" | `set_updated_at` | `BEFORE UPDATE` | `NOW()`, `updated_at` |
| "Rellenar fecha de creación" | `set_created_at` | `BEFORE INSERT` | `created_at`, `NOW()` |
| "Contar cuántas veces se editó" | `incrementar_contador` | `BEFORE UPDATE` | `COALESCE(OLD.x, 0) + 1` |
| "Guardar log/historial en otra tabla" | `log_xxx` | `AFTER INSERT/UPDATE` | `INSERT INTO logs`, `NEW.id` |
| "Validar que no sean negativos" | `validar_xxx` | `BEFORE INSERT OR UPDATE` | `RAISE EXCEPTION`, `IF NEW.x < 0` |
| "Calcular automáticamente un campo" | `calcular_xxx` | `BEFORE INSERT` | `NEW.campo := formula` |
| "Crear perfil al registrarse" | `handle_new_user` | `AFTER INSERT ON auth.users` | `SECURITY DEFINER`, `raw_user_meta_data` |

---

## 🐛 ERRORES COMUNES Y SOLUCIONES RÁPIDAS

| Error | Por qué pasa | Solución en 5 segundos |
|-------|-------------|------------------------|
| `function does not exist` | Creaste trigger antes que función | Ejecuta PASO 2 (función) ANTES del PASO 3 (trigger) |
| `relation "cursos" does not exist` | La tabla no existe | Crea la tabla primero o revisa el nombre |
| `column "updated_at" does not exist` | No ejecutaste el ALTER TABLE | Ejecuta PASO 1 antes de probar el trigger |
| `trigger already exists` | Lo creaste dos veces | Usa `CREATE OR REPLACE TRIGGER` o `DROP TRIGGER IF EXISTS` |
| `updated_at` no cambia | El trigger es AFTER en vez de BEFORE | Debe ser `BEFORE UPDATE`, no `AFTER UPDATE` |
| `updated_at` sigue NULL | El trigger no se disparó | Solo se dispara en UPDATE, no en INSERT. Haz un UPDATE de prueba |
| `null value violates not-null constraint` | Campo obligatorio sin valor | Usa `DEFAULT` o rellena en el trigger BEFORE INSERT |
| `new row violates check constraint` | El dato no cumple la regla CHECK | Revisa que el valor cumpla la condición del CHECK |
| `duplicate key value violates unique constraint` | Valor repetido en columna UNIQUE | Revisa que no exista ya ese valor |
| `violates foreign key constraint` | El ID referenciado no existe en la tabla padre | Inserta primero en la tabla padre |
| `permission denied for table auth.users` | Falta `SECURITY DEFINER` | Añade `SECURITY DEFINER` a la función |

---

## ⚡ COMANDOS RÁPIDOS (Copiar y pegar en el examen)

### El mínimo indispensable (template ultra-rápido)
```sql
ALTER TABLE cursos ADD COLUMN IF NOT EXISTS updated_at timestamptz;
CREATE OR REPLACE FUNCTION set_updated_at() RETURNS trigger LANGUAGE plpgsql AS $$ BEGIN NEW.updated_at = NOW(); RETURN NEW; END; $$;
CREATE TRIGGER trigger_set_updated_at BEFORE UPDATE ON cursos FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

### Probar que funciona
```sql
SELECT id, nombre, updated_at FROM cursos LIMIT 1;
UPDATE cursos SET nombre='Prueba' WHERE id=1;
SELECT id, nombre, updated_at FROM cursos WHERE id=1;
```

### Borrar y recrear (si la lías)
```sql
DROP TRIGGER IF EXISTS trigger_set_updated_at ON cursos;
DROP FUNCTION IF EXISTS set_updated_at();
-- Luego vuelve a ejecutar el template
```

---

## 🎯 EJERCICIOS TIPO EXAMEN (Practica estos)

### Ejercicio 1: Trigger de auditoría
Crea un trigger que, cada vez que se actualice un curso, guarde en una tabla `auditoria_cursos` el `id`, el `nombre` anterior (`OLD.nombre`) y el nuevo (`NEW.nombre`), más la fecha.

<details>
<summary>Solución</summary>

```sql
CREATE TABLE IF NOT EXISTS auditoria_cursos (
  id serial PRIMARY KEY,
  curso_id int NOT NULL,
  nombre_anterior text,
  nombre_nuevo text,
  modificado_en timestamptz NOT NULL DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION auditar_cambio_nombre()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
  IF OLD.nombre IS DISTINCT FROM NEW.nombre THEN
    INSERT INTO auditoria_cursos (curso_id, nombre_anterior, nombre_nuevo)
    VALUES (NEW.id, OLD.nombre, NEW.nombre);
  END IF;
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_auditar_cambio
  AFTER UPDATE ON cursos
  FOR EACH ROW
  EXECUTE FUNCTION auditar_cambio_nombre();
```

**Nota**: `IS DISTINCT FROM` compara incluso con NULL (mejor que `<>`).

</details>

---

### Ejercicio 2: Función de clasificación
Crea una función `clasificar_curso(horas int)` que devuelva `'corto'` si < 20h, `'medio'` si 20-40h, `'largo'` si > 40h.

<details>
<summary>Solución</summary>

```sql
CREATE OR REPLACE FUNCTION clasificar_curso(horas int)
RETURNS text
LANGUAGE plpgsql
AS $$
DECLARE
  resultado text;
BEGIN
  IF horas < 20 THEN
    resultado := 'corto';
  ELSIF horas <= 40 THEN
    resultado := 'medio';
  ELSE
    resultado := 'largo';
  END IF;
  RETURN resultado;
END;
$$;

-- Probar:
SELECT clasificar_curso(15);  -- 'corto'
SELECT clasificar_curso(30);  -- 'medio'
SELECT clasificar_curso(50);  -- 'largo'
```

</details>

---

### Ejercicio 3: Trigger que impide borrar si tiene dependencias
Crea un trigger que impida borrar un curso si tiene alumnos matriculados (usando `RAISE EXCEPTION`).

<details>
<summary>Solución</summary>

```sql
CREATE OR REPLACE FUNCTION impedir_borrado_con_alumnos()
RETURNS trigger
LANGUAGE plpgsql
AS $$
DECLARE
  num_alumnos int;
BEGIN
  SELECT COUNT(*) INTO num_alumnos FROM matriculas WHERE curso_id = OLD.id;

  IF num_alumnos > 0 THEN
    RAISE EXCEPTION 'No se puede borrar el curso % porque tiene % alumnos matriculados', OLD.id, num_alumnos;
  END IF;

  RETURN OLD;  -- en BEFORE DELETE devolvemos OLD
END;
$$;

CREATE TRIGGER trigger_impedir_borrado
  BEFORE DELETE ON cursos
  FOR EACH ROW
  EXECUTE FUNCTION impedir_borrado_con_alumnos();
```

</details>

---

## ✅ CHECKLIST FINAL ANTES DE ENTREGAR (SQL)

- [ ] La función se creó ANTES que el trigger (orden correcto)
- [ ] El `ALTER TABLE` se ejecutó antes de crear el trigger
- [ ] El trigger es `BEFORE UPDATE` (no AFTER) si modifica `NEW`
- [ ] La función devuelve `RETURN NEW` (o `RETURN OLD` en DELETE)
- [ ] La función tiene `RETURNS trigger` y `LANGUAGE plpgsql`
- [ ] Usé `CREATE OR REPLACE` para evitar errores de "ya existe"
- [ ] Demostré que funciona con `SELECT` → `UPDATE` → `SELECT`
- [ ] Si uso `auth.users`, la función tiene `SECURITY DEFINER`
- [ ] Si valido datos, uso `RAISE EXCEPTION` con mensaje claro
- [ ] Si uso contadores, uso `COALESCE(OLD.x, 0)` para evitar NULL

---

> 💡 **Tip final**: En el examen, abre este archivo, copia el template base, busca y reemplaza `'cursos'` por tu tabla, ejecuta, demuestra con las queries de prueba. **5 minutos y tienes la tarea completa.**

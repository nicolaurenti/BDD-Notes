---
tags:
  - tp3
---

# CHECK y ASSERTION

## CHECK y lógica de tres valores

```sql
ALTER TABLE articulo
ADD CONSTRAINT ck_nacionalidad
CHECK (nacionalidad IN ('Argentina', 'Español', 'Inglesa', 'Alemana', 'Chilena'));

ALTER TABLE articulo
ADD CONSTRAINT ck_fecha_publicacion
CHECK (fecha_pub >= DATE '2010-01-01');

ALTER TABLE producto
ADD CONSTRAINT ck_descripcion_presentacion
CHECK (descripcion IS NOT NULL OR presentacion IS NOT NULL);
```

Un `CHECK` se incumple cuando da **FALSE**. Si da `UNKNOWN` por un nulo, no lo rechaza: agregar `NOT NULL` cuando la información sea obligatoria. Diferencia importante con `WHERE`, que solo conserva `TRUE`.

Una implicación «si A, entonces B» se expresa como `NOT A OR B`:

```sql
-- Después de 2020: desde 2021 inclusive.
CHECK (fecha_pub < DATE '2021-01-01' OR nacionalidad = 'Argentina')
```

Si la nacionalidad debe estar informada siempre, agregar `NOT NULL`. Si solo es obligatoria para artículos posteriores a 2020, la segunda rama debe incluir `nacionalidad IS NOT NULL`.

Para un prefijo literal `S_`:

```sql
CHECK (cod_suc LIKE 'S!_%' ESCAPE '!')
```

El `_` sin escape significa «cualquier carácter». Ver Manejo de strings.

## Restricciones con conteos y ASSERTION

Patrón: **no debe existir un caso infractor**.

```sql
-- SQL estándar del ejercicio; PostgreSQL no admite CREATE ASSERTION.
CREATE ASSERTION max_15_palabras
CHECK (NOT EXISTS (
    SELECT id_articulo
    FROM contiene
    GROUP BY id_articulo
    HAVING COUNT(*) > 15
));
```

Se supone una fila de `contiene` por palabra clave asociada. Si el modelo permite repeticiones, definir qué se cuenta y usar la clave completa correspondiente.

Para las horas dentro de la tarea, el caso infractor es:

```sql
SELECT v.nro_voluntario
FROM voluntario v
JOIN tarea t ON t.id_tarea = v.id_tarea
WHERE v.horas_aportadas < t.min_horas
   OR v.horas_aportadas > t.max_horas;
```

La aserción exige que esa consulta esté vacía. Si los nulos están prohibidos, controlarlos además: las comparaciones ordinarias no detectan un nulo como infracción.

En PostgreSQL, un `CHECK` no admite una subconsulta y no garantiza controles sobre otras filas usando una función que las consulte. Elegir primero FK/UNIQUE cuando expresen la regla; para controles generales, ver TP4 - Triggers.

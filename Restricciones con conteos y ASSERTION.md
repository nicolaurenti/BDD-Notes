---
tags:
  - tp3
---

# Restricciones con conteos y ASSERTION


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

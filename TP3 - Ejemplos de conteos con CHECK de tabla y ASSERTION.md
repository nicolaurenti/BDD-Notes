---
tags:
  - tp3
---

# TP3 - Ejemplos de conteos con CHECK de tabla y ASSERTION

Planteos declarativos SQL-1999: máximo de palabras y horas dentro de los límites de tarea. Se indica lo que PostgreSQL no admite.

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

En el apartado PostgreSQL, no escribir subconsultas dentro de CHECK ni CREATE ASSERTION como si fueran recursos disponibles. Si FK o UNIQUE expresan la regla, usarlos; de lo contrario, indicar que no se puede implementar con los recursos declarativos vistos en este práctico.

## CHECK de tabla según SQL-1999

La cátedra permite plantear una subconsulta en un CHECK de tabla cuando el control necesita varias filas de esa tabla. Por ejemplo, para el límite de asociaciones:

```sql
-- Planteo declarativo SQL-1999; no es ejecutable en PostgreSQL.
ALTER TABLE contiene
ADD CONSTRAINT ck_max_palabras
CHECK (NOT EXISTS (
    SELECT id_articulo
    FROM contiene
    GROUP BY id_articulo
    HAVING COUNT(*) > 15
));
```

La consulta devuelve artículos infractores. Con 15 asociaciones no devuelve ese artículo; con 16, sí. NOT EXISTS invierte esa existencia para expresar la condición válida. La misma condición puede escribirse como ASSERTION, pero para clasificarla sigue siendo de tabla porque solo consulta CONTIENE.

Para «solo se pueden publicar artículos argentinos con hasta diez palabras», separar las posibles lecturas: exigir nacionalidad argentina a todos los artículos y, además, el máximo diez no es lo mismo que imponer el máximo únicamente cuando el artículo es argentino. Declarar la interpretación elegida antes de construir la negación de los infractores.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).

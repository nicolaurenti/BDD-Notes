---
tags:
  - tp7
---

# TP7 - Ejemplos de reescritura con conteos y existencia

Casos de tareas con más de cinco voluntarios e investigadores que participan en proyectos.

## Ejercicio 2: tareas con más de 5 voluntarios

Se puede quitar la tarea repetida dentro de la subconsulta:

```sql
SELECT t.*
FROM tarea t
WHERE t.id_tarea IN (
    SELECT v.id_tarea
    FROM voluntario v
    GROUP BY v.id_tarea
    HAVING COUNT(*) > 5
);
```

El conteo se realiza directamente sobre voluntario. Una subconsulta que devuelve un grupo con id NULL no lo hace coincidir con una PK de tarea no nula. Otra opción es un conteo correlacionado, pero no asegurar que sea más rápido: comparar planes.

## Ejercicio 3: joins innecesarios y DISTINCT

Para investigadores que trabajan en algún proyecto:

```sql
SELECT e.*
FROM empleado e
WHERE e.cargo = 'investigador'
  AND EXISTS (
      SELECT 1
      FROM trabaja_en te
      WHERE te.id_empleado = e.id_empleado
  );
```

- EMPLEADO es necesario para obtener sus datos y filtrar cargo.
- PROYECTO puede ser redundante si una FK garantiza la existencia del proyecto para cada asociación válida y no se filtra por atributos de proyecto. Si esa garantía no existe o la FK puede ser nula y representa una asociación incompleta, revisar la equivalencia.
- DISTINCT en el join original elimina repeticiones producidas por varios proyectos, pero tiene un costo posible de ordenamiento o hash. EXISTS expresa directamente una condición de existencia.

Para proyectos llamados IA con empleados, aplicar el mismo patrón desde PROYECTO y filtrar su nombre, correlacionando con TRABAJA_EN. El índice cuyo prefijo sea `cod_proyecto` puede favorecer esa búsqueda.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).

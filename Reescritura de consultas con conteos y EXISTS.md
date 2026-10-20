---
tags:
  - tp7
---

# Reescritura de consultas con conteos y EXISTS

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

## Ejercicio 4: más de dos grupos exclusivos

Antes de quitar GRUPO o el filtro `tipo_gr = 'E'`, verificar qué restricciones garantizan que toda fila de GR_EXCLUSIVO representa realmente un grupo exclusivo.

Si esa garantía existe y cada pertenencia es única, puede contarse en una subconsulta:

```sql
SELECT u.tipo_usuario, u.cod_usuario, u.apellido, u.nombre
FROM usuario u
WHERE EXISTS (
    SELECT 1
    FROM integra i
    JOIN gr_exclusivo ge ON ge.nro_grupo = i.nro_grupo
    WHERE i.tipo_usuario = u.tipo_usuario
      AND i.cod_usuario = u.cod_usuario
    GROUP BY i.tipo_usuario, i.cod_usuario
    HAVING COUNT(*) > 2
);
```

Sin unicidad de la pertenencia, evaluar `COUNT(DISTINCT i.nro_grupo)` para contar grupos. No confundir «más de 2 pertenencias» con «más de 2 grupos». Las columnas de la clave del usuario deben compararse juntas.

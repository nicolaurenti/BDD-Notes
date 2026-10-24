---
tags:
  - tp5
---

# Vistas con agregados


Una vista de cantidades no dice cómo repartir una actualización sobre las filas que la producen. Cambiar `cantidad_voluntarios` de 4 a 6 no identifica qué dos personas incorporar.

```sql
CREATE VIEW institucion_cantidad_voluntarios AS
SELECT i.id_institucion,
       i.nombre_institucion,
       COUNT(v.nro_voluntario) AS cantidad_voluntarios
FROM institucion i
LEFT JOIN voluntario v ON v.id_institucion = i.id_institucion
GROUP BY i.id_institucion, i.nombre_institucion;
```

No es automáticamente actualizable. Se puede decidir permitir modificar solo el nombre mediante un trigger y rechazar cambios del conteo. No inventar personas para justificar una escritura sobre el agregado.

Para cantidades históricas, distinguir **registros históricos** (`COUNT(*)`) de **tareas diferentes** (`COUNT(DISTINCT id_tarea)`) según el enunciado. Para instituciones con cero voluntarios, usar LEFT JOIN y contar la PK del voluntario, no `COUNT(*)`. El conteo de la PK del lado opcional evita contar la fila extendida con nulos.

## Por qué no hay una escritura automática única

Una institución con voluntarios A y B tiene cantidad 2. Pedir que la vista pase a cantidad 3 no dice qué persona agregar, sus horas, su tarea ni sus demás atributos obligatorios. Pedir que un promedio pase de 10 a 15 tampoco define cuál de los valores originales cambia. Esa falta de una traducción única explica la no actualizabilidad automática del agregado.

En el ejemplo, sí se puede diseñar un INSTEAD OF UPDATE que permita cambiar nombre_institucion en INSTITUCION y rechace todo cambio de cantidad_voluntarios. Eso habilita una operación concreta, no vuelve escribible el conteo por sí mismo.

La elección del conteo también forma parte de la definición: en una institución con historial de tareas A, A y B, hay tres períodos pero dos tareas diferentes. COUNT(DISTINCT id_tarea) responde por tareas; COUNT de la clave del registro responde por períodos. Si la consigna pide todas las instituciones, comenzar por INSTITUCION y conservar las que no tienen coincidencias con LEFT JOIN.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).

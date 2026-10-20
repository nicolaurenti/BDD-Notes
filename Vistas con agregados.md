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

Para cantidades históricas, distinguir **registros históricos** (`COUNT(*)`) de **tareas diferentes** (`COUNT(DISTINCT id_tarea)`) según el enunciado. Para instituciones con cero voluntarios, usar LEFT JOIN y contar la PK del voluntario, no `COUNT(*)`. Ver Conteos con joins.

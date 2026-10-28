---
tags:
  - tp7
---

# 2. Reescrituras equivalentes y comparación de planes

## Idea central

Reescribir es expresar el mismo pedido con otro SQL para facilitar una ejecución eficiente. Primero se demuestra que conserva el resultado y después se comprueba si mejora la ejecución.

## Cómo razonarlo

Las filminas usan álgebra relacional: selección σ filtra filas, proyección π conserva atributos y ensamble ⋈ relaciona filas. Las heurísticas principales son adelantar filtros, aplicar primero los más restrictivos y transportar solo columnas necesarias. Hay que conservar también las columnas que todavía se necesitan para unir, filtrar o agrupar.

Por ejemplo, `σcargo='investigador'(E ⋈ T)` puede transformarse en `(σcargo='investigador'(E)) ⋈ T` porque el filtro depende solo de E. Para inner joins, conmutatividad y asociatividad permiten explorar diferentes órdenes conservando sus condiciones. No extender estas reglas sin más a outer joins, agregaciones o LIMIT.

La equivalencia SQL conserva columnas, valores, repeticiones, comportamiento ante NULL y el orden explícitamente solicitado. El álgebra clásica trabaja con conjuntos; SQL conserva duplicados salvo operaciones que los eliminan. Una transformación algebraica requiere revisar esa diferencia.

## Ejemplo

En vez de unir tarea con voluntario dentro de una subconsulta para contar, puede agruparse voluntario directamente:

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

Con id_tarea como PK de tarea y nro_voluntario como PK de voluntario, el conteo coincide con COUNT(v.nro_voluntario) del join original. La consulta exterior recupera las tareas existentes. La resolución explica ambas alternativas y sus planes.

Para comparar, usar los mismos datos, índices y configuración: EXPLAIN muestra la predicción; EXPLAIN ANALYZE aporta evidencia de ejecución. Revisar accesos, filas intermedias, loops y tiempo total. Repetir alternando el orden reduce la influencia de caché y carga.

En datos de prueba, `A EXCEPT ALL B` y `B EXCEPT ALL A` deben quedar vacíos. Esto comprueba diferencias y multiplicidades sobre esos datos, no prueba equivalencia universal ni compara el orden.

## Errores frecuentes

Menos SQL no implica menos trabajo. Dos escrituras pueden producir el mismo plan. Cambiar IN por JOIN puede introducir duplicados; cambiar NOT IN por NOT EXISTS puede cambiar el efecto de NULL. Una ejecución más rápida no demuestra que siempre lo será.

## Qué recordar

**Corrección → propuesta → plan → medición → conclusión limitada a la evidencia.** [Reglas de expresiones de tablas](https://www.postgresql.org/docs/15/queries-table-expressions.html).

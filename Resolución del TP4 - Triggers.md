---
tags:
  - tp4
---

# Resolución del TP4 - Triggers

Resolución de los ejercicios **1, 2 y 3**, con grillas, implementaciones, pruebas y resultados. El ejercicio 2 parte de tu borrador: se corrigió el segundo `ON entrega` para actuar sobre `renglon_entrega`; la versión previa permanece en el historial Git.

## Ejercicios resueltos

- [[TP4 - Resolución del ejercicio 1|1 — Restricciones procedurales del TP3]].
- [[TP4 - Resolución del ejercicio 2|2 — Auditoría de ENTREGA y RENGLON_ENTREGA]].
- [[TP4 - Resolución del ejercicio 3|3 — Simulación de sueldos ROW y STATEMENT]].

## Para probar en DataGrip

Antes de ejecutar SQL, comprobar conexión y esquema:

```sql
SELECT current_database(), current_schema();
```

Artículos usa las tablas del TP3; Voluntarios y Películas son bases distintas con esquema `public`. No ejecutar todos los scripts en una misma conexión. Las tablas y triggers se instalan una sola vez en un esquema de práctica con el modelo base ya creado. Las variantes ROW y STATEMENT del ejercicio 2 son alternativas: no se instalan juntas.

Los scripts están también en `/Users/nicolaurenti/DataGripProjects/FinalBases/resoluciones_tp4`. El SQL completo queda incluido en las notas hijas para leerlo desde la tablet.

## Validación

**65 casos distintos pasaron en una instancia temporal aislada de PostgreSQL.** Se comprobaron base, esquema y directorio de datos antes de cada lote. Se usaron datos controlados, no las instancias existentes del proyecto. Las pruebas de modificación se revirtieron; tus bases habituales no se modificaron.

Las comprobaciones se hicieron secuencialmente. Los identificadores de auditoría usan `MAX + 1`, sin secuencias, con ese supuesto. Se mantiene el manejo de NULL de las comparaciones declarativas, salvo la política de cambios de institución que se declara expresamente.

## Pendientes

Los ejercicios 4, 5 y 6 del TP4 quedan pendientes.

## Material

`../Practicos/Practico 4 - Acciones Procedurales.docx` y `Practico 3 - Restricciones de Integridad Declarativas.docx`. Filminas `BDI_04 (SQL Procedural).pdf` de la cátedra.

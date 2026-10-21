---
tags:
  - tp7
---

# Reescritura de consultas con conteos y EXISTS

## Reescrituras de cantidad y existencia

Quitar una unión o cambiarla por EXISTS exige justificar qué filas y repeticiones se conservan. La existencia no reemplaza una condición de cantidad mínima.

[[TP7 - Ejemplos de reescritura con conteos y existencia|Ejemplos de reescritura con conteos y existencia]]

## Reescritura sobre un subtipo

Antes de eliminar GRUPO o su filtro, comprobar qué restricciones garantizan la pertenencia al subtipo exclusivo y la unicidad de cada asociación.

[[TP7 - Ejemplo de usuarios en más de dos grupos exclusivos|Ejemplo de usuarios en más de dos grupos exclusivos]]

## Comprobar la equivalencia con duplicados

Si un investigador participa en tres proyectos, el join original produce tres filas de ese empleado. DISTINCT deja una. EXISTS también produce una porque la consulta exterior recorre EMPLEADO y la subconsulta solo verifica presencia de una asociación. Quitar DISTINCT y conservar el join produciría tres: no sería equivalente a la salida pedida.

Quitar PROYECTO solo es correcto si su existencia está garantizada por la FK apropiada y no se necesitan filtros ni datos suyos. Si se pide «participa en IA», el nombre pertenece a PROYECTO y no se puede eliminar ese control.

Para tareas con más de cinco voluntarios, agrupar directamente VOLUNTARIO evita reunir dentro de la subconsulta la tabla TAREA que se vuelve a consultar afuera. Pero no se puede reemplazar el conteo por un simple EXISTS: tener uno no equivale a tener más de cinco.

En grupos exclusivos, probar un usuario con tres pertenencias repetidas al mismo grupo y otro con tres grupos diferentes. Si el modelo permite la primera situación, COUNT(*) confunde pertenencias con grupos. COUNT(DISTINCT nro_grupo) o la garantía de unicidad de la asociación son lo que permite afirmar que se cuentan grupos distintos.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).

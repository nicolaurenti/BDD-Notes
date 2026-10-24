---
tags:
  - tp5
---

# TP5 - Ejemplo de selección y migración de filas

Se comparan envíos de 700 y 300 unidades y un UPDATE que intenta cambiar su visibilidad.

## Diferenciar selección y comprobación posterior

Supongamos ENVIO con `(P1, A1, 700)` y `(P2, A2, 300)`. ENVIOS500 ve solo la primera. Un UPDATE sobre ENVIOS500 que busque P2 no encuentra la segunda fila, aunque proponga llevarla a 1000: el WHERE de la vista decide qué filas se pueden seleccionar antes de aplicar SET.

Si el UPDATE selecciona P1 y propone 100, sin WCO modifica la base y la fila deja de aparecer. Con WCO, se comprueba el resultado propuesto y se rechaza la sentencia porque 100 no satisface `cantidad >= 500`.

La «migración» no es un traslado a otra tabla: es dejar de pertenecer al resultado de la vista. La fila puede seguir existiendo en ENVIO. Para explicar el ejercicio, mostrar por separado el estado base y las filas visibles.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).

---
tags:
  - tp5
---

# Preservación de clave en vistas de ensamble


Una tabla conserva su clave en el ensamble si cada fila suya aparece como máximo una vez en el resultado. Analizar cardinalidades y restricciones, no solo que la clave figure en el SELECT.

- `ciudad JOIN pais` por su FK: cada ciudad tiene como máximo un país; la clave de ciudad se conserva. Un país puede repetirse por sus ciudades.
- `renglon_entrega JOIN pelicula`: cada renglón referencia una película. Se conserva la clave del renglón `(nro_entrega, codigo_pelicula)`; una película puede repetirse en muchos renglones.

En el análisis de cátedra, esto permite razonar qué tabla recibe una escritura sin ambigüedad. En PostgreSQL, esas vistas de join requieren un mecanismo explícito para escribir. Cambiar `titulo` desde un renglón podría afectar una película compartida por muchos renglones: definir esa política, no asumirla.

## Comprobar preservación con una instancia

Supongamos país AR con ciudades 1 y 2. Al unir CIUDAD con PAIS se obtienen `(1, AR)` y `(2, AR)`. Cada ciudad aparece una vez: su id sigue identificando la fila de la vista. El país AR aparece dos veces: su id ya no identifica una única fila del ensamble.

Según el criterio de las filminas, la tabla preservada es la que propaga su clave al resultado. En CIUDAD JOIN PAIS, modificar nombre_ciudad se puede asociar a una ciudad concreta. Modificar nombre_pais desde una fila de la vista afectaría al país compartido y cambiaría lo que se muestra en otras filas: no es el mismo tipo de propagación.

En RENGLON_ENTREGA JOIN PELICULA, suponer renglones `(10, P1)` y `(11, P1)`. La película aparece dos veces, pero cada clave de renglón una. La clave preservada es `(nro_entrega, codigo_pelicula)`; código_pelicula por sí solo no alcanza.

No decidir con la cantidad de filas que casualmente hay hoy. Si por ahora cada país tiene una ciudad, id_pais parece único en los datos, pero el modelo permite varias. La preservación debe deducirse de PK, FK y cardinalidades, no de una coincidencia temporal de la instancia.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).

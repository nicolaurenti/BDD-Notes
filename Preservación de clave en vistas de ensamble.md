---
tags:
  - tp5
---

# Preservación de clave en vistas de ensamble


Una tabla conserva su clave en el ensamble si cada fila suya aparece como máximo una vez en el resultado. Analizar cardinalidades y restricciones, no solo que la clave figure en el SELECT.

- `ciudad JOIN pais` por su FK: cada ciudad tiene como máximo un país; la clave de ciudad se conserva. Un país puede repetirse por sus ciudades.
- `renglon_entrega JOIN pelicula`: cada renglón referencia una película. Se conserva la clave del renglón `(nro_entrega, codigo_pelicula)`; una película puede repetirse en muchos renglones.

En el análisis de cátedra, esto permite razonar qué tabla recibe una escritura sin ambigüedad. En PostgreSQL, esas vistas de join requieren un mecanismo explícito para escribir. Cambiar `titulo` desde un renglón podría afectar una película compartida por muchos renglones: definir esa política, no asumirla.

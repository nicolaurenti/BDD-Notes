---
tags:
  - tp7
---

# EXPLAIN y EXPLAIN ANALYZE

```sql
EXPLAIN
SELECT * FROM pelicula WHERE genero = 'TERROR';

EXPLAIN ANALYZE
SELECT * FROM pelicula WHERE genero = 'TERROR';
```

- `EXPLAIN` muestra el plan estimado, sin ejecutar esa consulta de datos.
- `ANALYZE` la ejecuta y agrega mediciones reales.

Los ejemplos usan SELECT. Un EXPLAIN ANALYZE de una escritura sí ejecuta sus efectos.

Un plan es un árbol: los nodos consumen resultados de sus hijos. El costo estimado no está en milisegundos. `rows` estima filas emitidas por el nodo, no necesariamente todas las inspeccionadas. En nodos repetidos, `actual rows` y tiempos suelen mostrarse por ejecución; considerar `loops`. No sumar tiempos inclusivos de todos los nodos como si fueran independientes.

## Leer una línea y su lugar en el árbol

Una salida ilustrativa de EXPLAIN es:

```text
Seq Scan on pelicula  (cost=0.00..25.00 rows=8 width=120)
  Filter: (genero = 'TERROR')
```

Indica lectura secuencial y filtrado por género. El primer costo es el necesario antes de producir filas; el segundo, el costo total estimado del nodo. rows=8 estima ocho filas de salida y width=120 su ancho promedio en bytes. No dice que se hayan leído solo ocho filas ni que el tiempo sea 25 milisegundos.

Con ANALYZE se añaden valores como:

```text
(actual time=0.010..0.080 rows=6 loops=1)
```

Ahora hay una medición: seis filas emitidas por esa ejecución. Una diferencia grande entre rows estimadas y reales puede explicar por qué el optimizador eligió una estrategia poco adecuada. Si un nodo interior tiene loops=100, su trabajo se repitió cien veces; mirar solo el tiempo promedio de una iteración oculta el esfuerzo acumulado.

Para leer un plan de join, empezar por las hojas: de dónde salen las filas y dónde se filtran. Luego explicar cómo cada padre reúne o agrupa esos resultados. Los tiempos de un padre pueden incluir los de sus hijos; sumarlos todos contaría trabajo repetido. Las líneas de esta sección son ejemplos de lectura, no planes obtenidos de las bases del proyecto.

## Análisis de planes y fechas del TP7

En 1.a, revisar el acceso a película por género y las claves de entrega/renglón. En 1.b, comparar versiones IN, EXISTS o JOIN con la misma semántica histórica, enlazando el voluntario con el historial por su identificador.

El archivo del TP7 tiene una discrepancia: el texto de 1.b menciona 01/01/2023, pero su SQL usa 24/07/1998. Elegir y declarar la fecha antes de comparar planes: cambiar el filtro puede cambiar mucho la selectividad.

### Qué explicar de cada plan

En 1.a, identificar primero dónde se aplica `genero = 'TERROR'`. Después seguir cómo las películas se vinculan a RENGLON_ENTREGA por código y cómo esos renglones alcanzan ENTREGA por nro_entrega. Si el filtro reduce mucho las películas, explicar cómo eso cambia el trabajo de las uniones siguientes. No afirmar que debe aparecer un tipo de join sin haber leído el plan real.

En 1.b, cada voluntario debe tener **algún registro histórico** cuya fecha cumpla el límite y cuya tarea tenga diferencia máxima–mínima de hasta 5000. Cambiar la fecha o consultar la tarea actual del voluntario cambia la pregunta. Una versión JOIN puede repetir a quien tiene varias coincidencias históricas; IN o EXISTS expresan existencia y evitan esa repetición exterior.

La fecha es una discrepancia de la fuente, no un detalle de formato. Para comparar dos versiones, usar en ambas 01/01/2023 si se sigue el texto, o 24/07/1998 si se analiza literalmente el SQL impreso, y dejarlo indicado. Compararlas con fechas distintas mezclaría reescritura con selectividad.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).

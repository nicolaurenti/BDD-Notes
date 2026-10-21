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

## Fuente de estudio

Enunciado del TP7 en la carpeta `../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).

---
tags:
  - tp7
---

# 1. Planes de ejecución

## Idea central

SQL expresa el resultado solicitado. El **plan de ejecución** describe las operaciones elegidas para obtenerlo: acceder a tablas, filtrar, unir, agrupar, eliminar duplicados y ordenar. El optimizador busca una estrategia razonablemente eficiente; no garantiza encontrar la mejor entre todas las posibles.

## Cómo razonarlo

La cátedra distingue dos técnicas que se combinan:

- **Heurísticas:** transformar la consulta para reducir trabajo, por ejemplo filtrar pronto y evitar productos cartesianos innecesarios.
- **Estimación de costos:** comparar estrategias según estadísticas y estructuras disponibles.

Para estimar, interesan cantidad de filas y bloques, tamaño de filas, valores distintos, distribución de valores e índices existentes. Cardinalidad puede significar cantidad de filas de una relación; cuando hablamos de la cardinalidad de una columna nos referimos a cuántos valores distintos contiene. Conviene indicar cuál de ambos sentidos se usa.

| Herramienta | Qué permite observar |
|---|---|
| EXPLAIN SELECT ... | Plan y estimaciones sin ejecutar el SELECT. |
| EXPLAIN ANALYZE SELECT ... | Plan más mediciones obtenidas al ejecutar el SELECT. |
| ANALYZE tabla | Actualiza estadísticas; es un comando distinto de EXPLAIN ANALYZE. |

Para leer un árbol, seguir las filas desde sus nodos de acceso hacia los padres. La indentación expresa dependencias. No significa que cada nodo termine antes de que empiece su padre: algunas operaciones entregan filas progresivamente.

## Ejemplo

```sql
EXPLAIN
SELECT titulo
FROM pelicula
WHERE genero = 'TERROR'
ORDER BY titulo;
```

Plan **ilustrativo**, con números inventados:

```text
Sort  (cost=12.93..12.98 rows=20 width=40)
  Sort Key: titulo
  -> Seq Scan on pelicula  (cost=0.00..12.50 rows=20 width=40)
       Filter: (genero = 'TERROR')
```

El recorrido produce películas de terror y Sort ordena esas coincidencias por título. Si hubiera cien películas y veinte coincidieran, el recorrido examinaría cien y el ordenamiento recibiría veinte.

- **cost=a..b:** costo inicial y total estimados, en unidades del optimizador, no milisegundos.
- **rows:** filas estimadas de salida del nodo, no necesariamente todas las examinadas.
- **width:** tamaño medio estimado de la fila de salida en bytes.

Con ANALYZE podría agregarse `actual time=0.010..0.040 rows=18 loops=1`: primera fila a 0.010 ms, finalización a 0.040 ms, dieciocho filas reales y una ejecución del nodo. Si aparecieran 82 filas descartadas por el filtro, se habrían examinado cien. Con varios loops, filas y tiempos se muestran como promedios por ejecución; rows=3 y loops=10 equivale aproximadamente a treinta filas emitidas en total.

Planning Time corresponde a planificar y Execution Time a la ejecución instrumentada. La instrumentación agrega sobrecarga. Index Cond señala una condición usada para acceder por el índice; Filter comprueba una condición sobre las filas examinadas por el nodo.

| Nodo habitual | Trabajo que representa |
|---|---|
| Seq Scan | Recorrido de la tabla. |
| Index Scan | Acceso mediante índice y búsqueda de datos en la tabla. |
| Index Only Scan | Puede obtener los valores del índice; aún puede consultar la tabla para verificar visibilidad. |
| Bitmap Index Scan + Bitmap Heap Scan | Obtención de ubicaciones desde índices y acceso por páginas de la tabla. |
| Nested Loop | Procesar la otra entrada para cada fila de una entrada; puede aprovechar índices. |
| Hash Join | Construir una estructura hash y buscar coincidencias de la otra entrada. |
| Merge Join | Combinar entradas ordenadas por las claves de unión. |
| Sort | Ordenar filas. |
| Aggregate / HashAggregate / GroupAggregate | Calcular agregados y, cuando corresponde, agrupar. |
| Unique | Eliminar duplicados de una entrada ordenada. |

## Errores frecuentes

Sumar los costos totales de todos los nodos cuenta trabajo repetido: un padre ya incluye el de sus hijos. Lo mismo impide sumar indiscriminadamente sus tiempos. Tampoco un Seq Scan demuestra un problema: para una tabla pequeña puede ser conveniente. Una diferencia grande entre filas estimadas y reales invita a revisar estadísticas y distribución, no demuestra por sí sola que falte un índice.

## Qué recordar

El punto no consiste solo en escribir EXPLAIN: hay que describir el recorrido de las filas, distinguir estimaciones de mediciones y justificar la estrategia observada. [Referencia técnica](https://www.postgresql.org/docs/15/using-explain.html).

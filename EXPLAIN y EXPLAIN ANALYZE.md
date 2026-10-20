---
tags:
  - tp7
---

# EXPLAIN y EXPLAIN ANALYZE


```sql
EXPLAIN
SELECT * FROM pelicula WHERE genero = 'TERROR';

EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM pelicula WHERE genero = 'TERROR';
```

- `EXPLAIN` muestra el plan estimado, sin ejecutar esa consulta de datos.
- `ANALYZE` la ejecuta y agrega mediciones reales.
- `BUFFERS` ayuda a distinguir páginas encontradas en caché y páginas leídas, entre otros accesos.

Los ejemplos usan SELECT. Un EXPLAIN ANALYZE de una escritura sí ejecuta sus efectos.

Un plan es un árbol: los nodos consumen resultados de sus hijos. El costo estimado no está en milisegundos. `rows` estima filas emitidas por el nodo, no necesariamente todas las inspeccionadas. En nodos repetidos, `actual rows` y tiempos suelen mostrarse por ejecución; considerar `loops`. No sumar tiempos inclusivos de todos los nodos como si fueran independientes. [Lectura de EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html).

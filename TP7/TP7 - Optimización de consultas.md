---
tags:
  - tp7
---

# TP7 - Optimización de consultas

Apunte construido tema por tema a partir de la conversación. Los ejemplos son ilustrativos: no se ejecutó SQL ni se obtuvieron planes de las bases locales.

## 1. Planes de ejecución

### Qué es un plan

SQL expresa qué resultado queremos. El plan de ejecución describe las operaciones que el motor elige para obtenerlo: acceder a tablas, filtrar, unir, ordenar o agrupar.

Por ejemplo, para buscar las películas de terror:

```sql
SELECT titulo
FROM pelicula
WHERE genero = 'TERROR';
```

El motor podría recorrer la tabla y comprobar el género de cada película. Si existe un índice adecuado, también podría usarlo para localizar coincidencias. La consulta expresa el mismo pedido; cambia el trabajo necesario para resolverlo. Sin ORDER BY, no se garantiza el orden de presentación.

### Qué decide el optimizador

El optimizador compara alternativas usando estimaciones de costo. Influyen el tamaño de las tablas, las estadísticas de los datos y los índices disponibles. No basta con escribir menos SQL para que una consulta sea más rápida.

### Cómo observarlo

```sql
EXPLAIN
SELECT titulo FROM pelicula WHERE genero = 'TERROR';
```

EXPLAIN muestra el plan estimado sin ejecutar la consulta. EXPLAIN ANALYZE sí la ejecuta y agrega mediciones reales. Para entender un plan conviene identificar primero los accesos a las tablas y después las operaciones que reciben esas filas; el plan tiene estructura de árbol.

Un fragmento inventado para practicar la lectura:

```text
Seq Scan on pelicula  (cost=0.00..12.50 rows=20 width=40)
  Filter: (genero = 'TERROR')
```

- Seq Scan: recorre la tabla.
- Filter: comprueba la condición.
- cost: costo inicial y total estimados, en unidades del optimizador, no milisegundos.
- rows: filas que se estima que saldrán de ese nodo, no todas las examinadas.
- width: tamaño medio estimado de cada fila de salida, en bytes.

Si hubiera 100 películas y solo 20 fueran de terror, este recorrido examinaría las 100 y devolvería 20. Esa diferencia ayuda a distinguir trabajo realizado de resultado obtenido. Los números del fragmento no describen nuestras tablas reales.

### Idea para retener

Una consulta indica el resultado; un plan explica cómo se intenta obtenerlo. Optimizar requiere comparar trabajo y preservar el resultado solicitado. Usar un índice no es automáticamente mejor: recorrer una tabla pequeña puede ser una elección razonable.

## Temas pendientes

2. Reescrituras equivalentes y comparación de planes.
3. Joins, subconsultas y ensambles innecesarios.
4. DISTINCT y eliminación de duplicados.
5. GROUP BY y HAVING.
6. Índices simples, compuestos y orden de columnas.
7. Índices ante igualdad, rangos, fechas, funciones y LIKE.
8. Árboles B+ y Bitmap.

## Material

Enunciado: Practicos/Practico 7 - Optimización de Consultas.docx, dentro de Bases de Datos, fuera del baúl. El práctico propone analizar planes y justificar índices.

Referencia técnica: [PostgreSQL — Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html).

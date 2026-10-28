---
tags:
  - tp7
---

# 3. Joins, subconsultas y ensambles innecesarios

## Idea central

Cada tabla debe aportar datos, filtros, comprobación de existencia, multiplicidad o una conexión necesaria. Que no aparezca en SELECT no demuestra que sobre.

## Cómo razonarlo

| Pedido | Forma natural de expresarlo |
|---|---|
| Datos de ambos lados o cada pareja coincidente | JOIN. |
| Filas con alguna coincidencia | EXISTS o IN. |
| Filas sin coincidencias | NOT EXISTS. |
| Relacionar con grupos ya calculados | JOIN con tabla derivada. |

Para quitar un inner join sin alterar filas, una condición suficiente es garantizar exactamente una coincidencia: FK válida y no nullable para existencia, y PK/UNIQUE de destino para unicidad. Deben comprobarse todas las columnas de una clave compuesta. Las restricciones deben cumplirse en el estado consultado.

Una tabla puente, un filtro suyo o una columna de salida impiden retirarla directamente. Si la FK admite NULL, el join excluye esas filas; al retirarlo puede ser necesario conservar `IS NOT NULL`.

## Ejemplo

Para investigadores que participan en algún proyecto:

```sql
SELECT e.*
FROM empleado e
WHERE e.cargo = 'investigador'
  AND EXISTS (
      SELECT 1 FROM trabaja_en t
      WHERE t.id_empleado = e.id_empleado
  );
```

Ana participa en dos proyectos y Bruno en uno: un join produce Ana, Ana y Bruno; EXISTS conserva una fila exterior por empleado que participa. SELECT 1 expresa existencia; no devuelve un 1 en la salida exterior. La referencia a e hace correlacionada la subconsulta, pero no obliga a ejecutar una búsqueda completa por empleado: el motor puede transformarla en un semijoin.

La alternativa con `e.id_empleado IN (SELECT t.id_empleado FROM trabaja_en t)` también expresa pertenencia sin multiplicar la fila exterior. Con NOT EXISTS se selecciona la ausencia. NOT IN exige revisar NULL; puede producir UNKNOWN cuando la subconsulta contiene uno.

Un LEFT JOIN más `WHERE columna_derecha IS NULL` expresa ausencia si esa columna no puede ser NULL en una coincidencia real. Una columna nullable ajena a la condición puede confundir «sin coincidencia» con «coincidencia cuyo dato es NULL».

## Errores frecuentes

No retirar proyecto si se filtra por su nombre. No retirar trabaja_en si es quien demuestra participación. No suponer que JOIN siempre supera a EXISTS o IN. Un semijoin o anti join es una estrategia posible, no un nodo garantizado. Cambiar comas por JOIN ... ON no fuerza por sí solo un orden físico ni una mejora.

## Qué recordar

Justificar lo que aporta cada tabla y qué garantiza que quitarla no cambie el resultado. [Subconsultas](https://www.postgresql.org/docs/current/functions-subquery.html) y [orden de joins](https://www.postgresql.org/docs/current/explicit-joins.html).

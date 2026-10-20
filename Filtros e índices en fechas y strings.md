---
tags:
  - tp7
---

# Filtros e índices en fechas y strings


Preferir comparar la columna directamente con un límite cuando la transformación sea equivalente:

```sql
-- Entregas posteriores al año 2010:
WHERE fecha_entrega >= DATE '2011-01-01'
```

Aplicar EXTRACT o una función a cada valor puede impedir usar un índice ordinario como condición de búsqueda. Un índice de expresión es otra opción si corresponde al patrón frecuente.

En el TP aparece `MONTHS_BETWEEN(SYSDATE, fecha_entrega)`: es sintaxis de Oracle. En PostgreSQL, una interpretación por antigüedad de calendario es:

```sql
WHERE fecha_entrega < CURRENT_DATE - INTERVAL '100 months'
```

No asumir equivalencia exacta con meses fraccionarios de Oracle en todos los bordes; indicar qué significa «más de 100 meses» y si se usan fechas o timestamps.

### Strings e igualdad

- `nombre = 'Juan Lucas' AND e_mail = 'Pla@gmail.com'`: evaluar selectividad y un índice compuesto; si el correo ya identifica casi una sola fila, su índice puede bastar.
- `nombre LIKE 'An%'`: un índice adecuado puede usar el prefijo; colación y clases de operadores importan.
- `nombre LIKE '%Maria'`: un B-tree ordinario no acota por prefijo. Se puede aprovechar la igualdad de e_mail y filtrar el nombre; para búsquedas de texto frecuentes hay alternativas como trigramas.
- `tipo = 'N'`: pocos valores distintos no implican automáticamente buena selectividad. Si casi todos son N, el índice puede no convenir.

PostgreSQL usa B-tree por defecto. El «bitmap» de un plan PostgreSQL no implica que se haya creado un índice bitmap persistente como en otros motores. La elección teórica de bitmap para atributos de baja cardinalidad también depende del volumen y las escrituras. [Tipos de índices](https://www.postgresql.org/docs/current/indexes-types.html).

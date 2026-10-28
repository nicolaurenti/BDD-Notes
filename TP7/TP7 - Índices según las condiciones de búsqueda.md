---
tags:
  - tp7
---

# 7. Índices según las condiciones de búsqueda

## Idea central

El índice debe corresponder a la operación que realiza el filtro. Importa tanto la columna como la expresión y el operador que se usan sobre ella.

## Cómo razonarlo

| Condición | Criterio inicial |
|---|---|
| columna = valor | B+ permite igualdad; valorar cuántas filas coinciden. |
| columna > valor / BETWEEN | B+ permite localizar y recorrer un intervalo ordenado. |
| nombre LIKE 'An%' | Hay prefijo fijo: un índice ordenado compatible puede ayudar. |
| nombre LIKE '%Maria' | No hay comienzo fijo; un B+ normal no delimita un rango de prefijo útil. |
| función(columna) = valor | Un índice sobre columna no equivale automáticamente a uno sobre la expresión. |
| varios filtros | Evaluar un compuesto o combinación de índices, sin duplicar estructuras innecesarias. |

Un filtro aprovechable como condición de acceso por índice suele llamarse **sargable**. Dejar la columna sin transformar suele facilitarlo, pero toda reescritura debe conservar el significado.

## Ejemplo

```sql
-- Intervalo de un año: la columna queda directamente comparada.
SELECT * FROM entrega
WHERE fecha_entrega >= DATE '2023-01-01'
  AND fecha_entrega < DATE '2024-01-01';
```

Expresa el año completo y puede usar un índice de fecha sin calcular el año para cada fila. Es preferible razonar con intervalos semiabiertos cuando existe hora, para no perder el último día.

Un índice de expresión es otra posibilidad si la expresión y el motor lo permiten:

```sql
CREATE INDEX idx_tarea_amplitud
ON tarea ((max_horas - min_horas));
```

Para `nombre LIKE 'An%'`, PostgreSQL puede necesitar una clase como varchar_pattern_ops según la intercalación. No es correcto prometer uso de cualquier índice textual. Para `'%Maria'`, si hay además igualdad por correo, este último puede localizar candidatos y luego comprobarse el sufijo.

El ejercicio usa MONTHS_BETWEEN y SYSDATE, funciones de Oracle. PostgreSQL tiene otras operaciones de fecha. Convertir «antigüedad de cien meses» a una fecha límite es útil, pero no prueba equivalencia exacta con MONTHS_BETWEEN en los límites: su cálculo contempla reglas de fin de mes y fracciones.

## Errores frecuentes

No indexar una expresión dependiente del día actual como si fuera inmutable: los datos del índice no se actualizarían solos al pasar el tiempo. No confundir «consulta frecuente» con «índice siempre rentable». SELECT * también exige recuperar muchas columnas: hallar ubicaciones no elimina el costo de devolver sus datos.

## Qué recordar

Revisar igualdad/rango/patrón, expresiones, selectividad y límites temporales. [Tipos y patrones](https://www.postgresql.org/docs/15/indexes-types.html), [expresiones](https://www.postgresql.org/docs/current/indexes-expressional.html) y [MONTHS_BETWEEN](https://docs.oracle.com/database/121/SQLRF/functions114.htm).

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
- `nombre LIKE '%Maria'`: un B-tree ordinario no acota por prefijo. Se puede aprovechar la igualdad de e_mail y filtrar el nombre; el prefijo del nombre no aporta un tramo inicial que buscar en ese B-tree.
- `tipo = 'N'`: pocos valores distintos no implican automáticamente buena selectividad. Si casi todos son N, el índice puede no convenir.

PostgreSQL usa B-tree por defecto. El «bitmap» de un plan PostgreSQL no implica que se haya creado un índice bitmap persistente como en otros motores. La elección teórica de bitmap para atributos de baja cardinalidad también depende del volumen y las escrituras.

## Responder el ejercicio de índices sin cambiar el dialecto

El ejercicio 5 escribe MONTHS_BETWEEN y SYSDATE; conservar esa expresión al analizar la consulta tal como está en la consigna. Su propósito es filtrar antigüedad de entregas. Si además se quiere ejecutarla en PostgreSQL, presentar la traducción de fechas por separado y declarar cómo se interpreta el límite; no reemplazar silenciosamente el SQL original.

| Condición del ejercicio | Razonamiento sobre el acceso |
|---|---|
| id_distribuidor > 50 | Un B+ permite rangos; la conveniencia depende de cuántos distribuidores superan 50 |
| tipo = 'N' | Evaluar distribución y baja cardinalidad; no afirmar que un índice siempre ayuda |
| nombre = ... AND e_mail = ... | Un compuesto puede servir; un correo muy selectivo puede bastar como primer acceso |
| nombre LIKE 'An%' y correo igual | El prefijo del nombre puede acotar; comparar con el acceso por correo |
| nombre LIKE '%Maria' y correo igual | El sufijo no fija el comienzo del nombre; el correo sigue siendo una igualdad aprovechable |

## B+ y bitmap en el contexto de la consigna

El TP plantea árboles B+ como estructura habitual e invita a considerar bitmap. Un bitmap representa para cada valor qué filas lo poseen; puede ser útil en atributos de pocos valores distintos y consultas que combinan condiciones, especialmente cuando predominan lecturas. TIPO es el candidato conceptual de la lista por su baja cardinalidad. Esa elección se justifica con volumen, distribución y carga de modificaciones, no solo con que el atributo sea un carácter.

Distinguir ese **índice bitmap** de un Bitmap Index Scan en PostgreSQL: este último es una estrategia del plan que puede usar índices ordinarios para reunir ubicaciones de filas. Ver un nodo Bitmap no demuestra que se creó el tipo de índice teórico del enunciado.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).

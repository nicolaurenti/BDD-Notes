---
tags:
  - tp7
---

# Índices y orden de columnas

Ejemplos de propuestas, no instrucciones ejecutadas:

```sql
CREATE INDEX idx_historico_voluntario
ON historico (nro_voluntario);

CREATE INDEX idx_entrega_distribuidor_fecha
ON entrega (id_distribuidor, fecha_entrega);
```

Un índice puede acelerar búsqueda, join u ordenamiento, a cambio de espacio y costo de INSERT/UPDATE/DELETE. PK y UNIQUE normalmente crean índices propios en PostgreSQL; una FK no crea automáticamente un índice en la tabla que referencia.

## Índices multicolumna

En un B-tree `(id_empleado, cod_proyecto)`, las condiciones sobre la columna inicial suelen permitir acotar mejor la búsqueda. Para consultar por proyecto suele ser más conveniente `(cod_proyecto, id_empleado)`.

Ambas combinaciones pueden expresar la misma unicidad, pero el orden físico sirve a accesos distintos. Para el análisis pedido por la cátedra, justificar con el prefijo del índice y las condiciones de búsqueda, sin basar la respuesta en optimizaciones de versiones posteriores.

## Mismo conjunto de columnas, distinto orden de búsqueda

En `(id_empleado, cod_proyecto)`, las asociaciones de un empleado quedan próximas entre sí; sirve para buscar los proyectos del empleado 3. En `(cod_proyecto, id_empleado)`, quedan próximas las asociaciones del proyecto 2; sirve para buscar sus empleados. Ambas PK impiden repetir la misma pareja, pero sus índices ordenan primero por componentes diferentes.

Para `id_distribuidor = 20 AND fecha_entrega >= ...`, el índice `(id_distribuidor, fecha_entrega)` permite localizar primero el distribuidor y luego el tramo de fechas. Si la consulta solo filtra fecha, esa disposición ya no ofrece el mismo prefijo útil. Hay que relacionar el orden con las consultas frecuentes, no con el orden alfabético de los nombres.

## Selectividad y costo de mantenimiento

Si una condición deja 10 filas de 100000, elimina gran parte de la tabla y puede favorecer un acceso por índice. Si deja 90000, quizá convenga una lectura secuencial. Un atributo con dos valores no es selectivo automáticamente: depende de cuántas filas satisfacen el valor buscado.

Cada inserción, baja o cambio de clave indexada también debe mantener el índice. Para proponer uno en el ejercicio 5, indicar qué consulta lo aprovecha, por qué reduce trabajo y si el mismo prefijo sirve a otras. No crear un índice por cada columna sin analizar el conjunto.

## Filtros e índices en fechas y strings

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

### Aplicación a las consultas del TP

Para proponer un índice se relacionan predicados, selectividad y estructura. El ejercicio conserva sus expresiones de fecha y pide considerar B+ y bitmap según el contexto.

#### Casos de elección de índices del ejercicio 5

Comparación de rangos, igualdad, prefijos y sufijos de texto; incluye la distinción entre un índice bitmap teórico y un nodo Bitmap del plan.

#### Responder el ejercicio de índices sin cambiar el dialecto

El ejercicio 5 escribe MONTHS_BETWEEN y SYSDATE; conservar esa expresión al analizar la consulta tal como está en la consigna. Su propósito es filtrar antigüedad de entregas. Si además se quiere ejecutarla en PostgreSQL, presentar la traducción de fechas por separado y declarar cómo se interpreta el límite; no reemplazar silenciosamente el SQL original.

| Condición del ejercicio | Razonamiento sobre el acceso |
|---|---|
| id_distribuidor > 50 | Un B+ permite rangos; la conveniencia depende de cuántos distribuidores superan 50 |
| tipo = 'N' | Evaluar distribución y baja cardinalidad; no afirmar que un índice siempre ayuda |
| nombre = ... AND e_mail = ... | Un compuesto puede servir; un correo muy selectivo puede bastar como primer acceso |
| nombre LIKE 'An%' y correo igual | El prefijo del nombre puede acotar; comparar con el acceso por correo |
| nombre LIKE '%Maria' y correo igual | El sufijo no fija el comienzo del nombre; el correo sigue siendo una igualdad aprovechable |

#### B+ y bitmap en el contexto de la consigna

El TP plantea árboles B+ como estructura habitual e invita a considerar bitmap. Un bitmap representa para cada valor qué filas lo poseen; puede ser útil en atributos de pocos valores distintos y consultas que combinan condiciones, especialmente cuando predominan lecturas. TIPO es el candidato conceptual de la lista por su baja cardinalidad. Esa elección se justifica con volumen, distribución y carga de modificaciones, no solo con que el atributo sea un carácter.

Distinguir ese **índice bitmap** de un Bitmap Index Scan en PostgreSQL: este último es una estrategia del plan que puede usar índices ordinarios para reunir ubicaciones de filas. Ver un nodo Bitmap no demuestra que se creó el tipo de índice teórico del enunciado.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).

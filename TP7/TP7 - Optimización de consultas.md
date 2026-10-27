---
tags:
  - tp7
---

# TP7 - Optimización de consultas

Este resumen desarrolla los ocho puntos acordados. Cada punto sigue la misma estructura: **idea central → cómo razonarlo → ejemplo → errores frecuentes → qué recordar**. Los ejemplos breves explican conceptos; la resolución de los ejercicios y los planes comprobados se encuentran en [[Resolución del TP7 - Optimización de consultas]].

## Recorrido de lectura

1. [Planes de ejecución](#1-planes-de-ejecución).
2. [Reescrituras equivalentes y comparación de planes](#2-reescrituras-equivalentes-y-comparación-de-planes).
3. [Joins, subconsultas y ensambles innecesarios](#3-joins-subconsultas-y-ensambles-innecesarios).
4. [DISTINCT y eliminación de duplicados](#4-distinct-y-eliminación-de-duplicados).
5. [Agrupamiento y conteos](#5-agrupamiento-y-conteos).
6. [Índices simples y compuestos](#6-índices-simples-y-compuestos).
7. [Índices según las condiciones de búsqueda](#7-índices-según-las-condiciones-de-búsqueda).
8. [Árboles B+ y Bitmap](#8-árboles-b-y-bitmap).

## 1. Planes de ejecución

### Idea central

SQL expresa el resultado solicitado. El **plan de ejecución** describe las operaciones elegidas para obtenerlo: acceder a tablas, filtrar, unir, agrupar, eliminar duplicados y ordenar. El optimizador busca una estrategia razonablemente eficiente; no garantiza encontrar la mejor entre todas las posibles.

### Cómo razonarlo

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

### Ejemplo

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

### Errores frecuentes

Sumar los costos totales de todos los nodos cuenta trabajo repetido: un padre ya incluye el de sus hijos. Lo mismo impide sumar indiscriminadamente sus tiempos. Tampoco un Seq Scan demuestra un problema: para una tabla pequeña puede ser conveniente. Una diferencia grande entre filas estimadas y reales invita a revisar estadísticas y distribución, no demuestra por sí sola que falte un índice.

### Qué recordar

El punto no consiste solo en escribir EXPLAIN: hay que describir el recorrido de las filas, distinguir estimaciones de mediciones y justificar la estrategia observada. [Referencia técnica](https://www.postgresql.org/docs/15/using-explain.html).

## 2. Reescrituras equivalentes y comparación de planes

### Idea central

Reescribir es expresar el mismo pedido con otro SQL para facilitar una ejecución eficiente. Primero se demuestra que conserva el resultado y después se comprueba si mejora la ejecución.

### Cómo razonarlo

Las filminas usan álgebra relacional: selección σ filtra filas, proyección π conserva atributos y ensamble ⋈ relaciona filas. Las heurísticas principales son adelantar filtros, aplicar primero los más restrictivos y transportar solo columnas necesarias. Hay que conservar también las columnas que todavía se necesitan para unir, filtrar o agrupar.

Por ejemplo, `σcargo='investigador'(E ⋈ T)` puede transformarse en `(σcargo='investigador'(E)) ⋈ T` porque el filtro depende solo de E. Para inner joins, conmutatividad y asociatividad permiten explorar diferentes órdenes conservando sus condiciones. No extender estas reglas sin más a outer joins, agregaciones o LIMIT.

La equivalencia SQL conserva columnas, valores, repeticiones, comportamiento ante NULL y el orden explícitamente solicitado. El álgebra clásica trabaja con conjuntos; SQL conserva duplicados salvo operaciones que los eliminan. Una transformación algebraica requiere revisar esa diferencia.

### Ejemplo

En vez de unir tarea con voluntario dentro de una subconsulta para contar, puede agruparse voluntario directamente:

```sql
SELECT t.*
FROM tarea t
WHERE t.id_tarea IN (
    SELECT v.id_tarea
    FROM voluntario v
    GROUP BY v.id_tarea
    HAVING COUNT(*) > 5
);
```

Con id_tarea como PK de tarea y nro_voluntario como PK de voluntario, el conteo coincide con COUNT(v.nro_voluntario) del join original. La consulta exterior recupera las tareas existentes. La resolución explica ambas alternativas y sus planes.

Para comparar, usar los mismos datos, índices y configuración: EXPLAIN muestra la predicción; EXPLAIN ANALYZE aporta evidencia de ejecución. Revisar accesos, filas intermedias, loops y tiempo total. Repetir alternando el orden reduce la influencia de caché y carga.

En datos de prueba, `A EXCEPT ALL B` y `B EXCEPT ALL A` deben quedar vacíos. Esto comprueba diferencias y multiplicidades sobre esos datos, no prueba equivalencia universal ni compara el orden.

### Errores frecuentes

Menos SQL no implica menos trabajo. Dos escrituras pueden producir el mismo plan. Cambiar IN por JOIN puede introducir duplicados; cambiar NOT IN por NOT EXISTS puede cambiar el efecto de NULL. Una ejecución más rápida no demuestra que siempre lo será.

### Qué recordar

**Corrección → propuesta → plan → medición → conclusión limitada a la evidencia.** [Reglas de expresiones de tablas](https://www.postgresql.org/docs/15/queries-table-expressions.html).

## 3. Joins, subconsultas y ensambles innecesarios

### Idea central

Cada tabla debe aportar datos, filtros, comprobación de existencia, multiplicidad o una conexión necesaria. Que no aparezca en SELECT no demuestra que sobre.

### Cómo razonarlo

| Pedido | Forma natural de expresarlo |
|---|---|
| Datos de ambos lados o cada pareja coincidente | JOIN. |
| Filas con alguna coincidencia | EXISTS o IN. |
| Filas sin coincidencias | NOT EXISTS. |
| Relacionar con grupos ya calculados | JOIN con tabla derivada. |

Para quitar un inner join sin alterar filas, una condición suficiente es garantizar exactamente una coincidencia: FK válida y no nullable para existencia, y PK/UNIQUE de destino para unicidad. Deben comprobarse todas las columnas de una clave compuesta. Las restricciones deben cumplirse en el estado consultado.

Una tabla puente, un filtro suyo o una columna de salida impiden retirarla directamente. Si la FK admite NULL, el join excluye esas filas; al retirarlo puede ser necesario conservar `IS NOT NULL`.

### Ejemplo

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

### Errores frecuentes

No retirar proyecto si se filtra por su nombre. No retirar trabaja_en si es quien demuestra participación. No suponer que JOIN siempre supera a EXISTS o IN. Un semijoin o anti join es una estrategia posible, no un nodo garantizado. Cambiar comas por JOIN ... ON no fuerza por sí solo un orden físico ni una mejora.

### Qué recordar

Justificar lo que aporta cada tabla y qué garantiza que quitarla no cambie el resultado. [Subconsultas](https://www.postgresql.org/docs/current/functions-subquery.html) y [orden de joins](https://www.postgresql.org/docs/current/explicit-joins.html).

## 4. DISTINCT y eliminación de duplicados

### Idea central

DISTINCT conserva una sola copia de cada combinación completa de valores seleccionados. No elimina «empleados repetidos» en abstracto: compara exactamente las columnas del SELECT.

### Cómo razonarlo

La PK de una tabla evita duplicados en ella, pero un join uno a muchos puede repetir sus columnas. Antes de retirar DISTINCT, demostrar que la proyección no se repetirá.

El motor puede eliminar duplicados ordenando y aplicando Unique o usando agrupamiento hash. Ambas alternativas consumen recursos; si el conjunto es grande pueden necesitar más memoria o trabajo en disco. Un índice o un orden ya disponible puede ayudar, pero no garantiza que desaparezca el trabajo.

### Ejemplo

Hay dos empleados diferentes llamados Ana; uno participa en dos proyectos y el otro en uno:

| SELECT sobre el join | Resultado |
|---|---|
| e.id_empleado, e.nombre | (1,Ana), (1,Ana), (2,Ana). |
| DISTINCT e.id_empleado, e.nombre | (1,Ana), (2,Ana). |
| DISTINCT e.nombre | Una sola fila Ana. |

Con EXISTS y `SELECT e.*`, una fila por empleado y su PK en la salida hacen innecesario DISTINCT. Si se devuelve solo nombre, EXISTS podría producir dos Ana: no es equivalente a DISTINCT nombre.

### Errores frecuentes

Agregar DISTINCT para ocultar un join mal planteado puede encubrir el error y agregar costo. Tampoco puede retirarse porque «hay claves primarias»: importa la multiplicidad después de unir y proyectar. Para eliminar duplicados, dos NULL se consideran iguales a estos efectos; no se aplica la lógica de `NULL = NULL` de un WHERE.

### Qué recordar

Preguntar **qué entidad quiero contar o devolver**, qué puede repetirla y si solo necesito existencia. DISTINCT tiene un significado y un costo, no es un arreglo automático. [Referencia técnica](https://www.postgresql.org/docs/15/queries-select-lists.html).

## 5. Agrupamiento y conteos

### Idea central

GROUP BY reúne filas que comparten claves; los agregados resumen cada grupo. WHERE filtra filas antes del agrupamiento; HAVING filtra grupos según lo calculado.

### Cómo razonarlo

Primero identificar la unidad: ¿personas, participaciones o grupos distintos? Después comprobar si los joins multiplican esa unidad.

| Expresión | Qué cuenta |
|---|---|
| COUNT(*) | Todas las filas del grupo. |
| COUNT(columna) | Filas donde esa columna no es NULL. |
| COUNT(DISTINCT columna) | Valores distintos no NULL de esa columna. |

Una restricción como «más de dos grupos exclusivos» significa tres o más; filtrar por tipo debe ocurrir antes del conteo. Las claves compuestas deben conservarse completas para no mezclar usuarios diferentes.

### Ejemplo

```sql
SELECT i.tipo_usuario, i.cod_usuario
FROM integra i
JOIN gr_exclusivo ge ON ge.nro_grupo = i.nro_grupo
GROUP BY i.tipo_usuario, i.cod_usuario
HAVING COUNT(*) > 2;
```

Si la PK de integra es (tipo_usuario,cod_usuario,nro_grupo) y gr_exclusivo tiene una fila por grupo, cada fila del join representa una pertenencia a un grupo exclusivo distinto. Un usuario con tres aparece; uno con dos no.

Puede agruparse esta relación estrecha primero y recién después unir con usuario para obtener apellido y nombre. Así se evita transportar nombres por todo el agrupamiento. Es una propuesta de reducción de trabajo, no una garantía de un plan más rápido.

Con LEFT JOIN, un padre sin hijos conserva una fila extendida con NULL: COUNT(*) da uno y COUNT(clave_hijo) da cero. Esa diferencia es importante para consultas que incluyen entidades sin relaciones.

### Errores frecuentes

Contar filas multiplicadas por otro join sobrecuenta entidades. COUNT(DISTINCT ...) puede expresar el conteo correcto, pero no sustituye analizar la causa. Mover condiciones de HAVING a WHERE solo es válido si conserva la semántica; una condición sobre el agregado no puede evaluarse sobre cada fila individual.

### Qué recordar

**Filtrar lo pertinente → contar la unidad correcta → aplicar el umbral → recuperar datos descriptivos.** [Referencia técnica](https://www.postgresql.org/docs/15/queries-table-expressions.html).

## 6. Índices simples y compuestos

### Idea central

Un índice es una estructura adicional para localizar filas sin recorrer necesariamente toda la tabla. Ocupa espacio y cuesta mantenerlo al insertar, borrar o modificar datos. Su utilidad depende de la carga de consultas y de la fracción de filas que evita examinar.

### Cómo razonarlo

Un índice simple usa una columna; uno compuesto usa varias. En un árbol B+ compuesto, las entradas se ordenan por la primera columna y, dentro de ella, por las siguientes: (a,b) no organiza los datos igual que (b,a).

Las condiciones de igualdad sobre el comienzo de la clave y el primer rango posterior suelen delimitar mejor la búsqueda. Condiciones posteriores aún pueden filtrarse en el índice, aunque no reduzcan del mismo modo el tramo recorrido. Este es el criterio básico del curso; no afirmar que una columna no inicial sea absolutamente inutilizable. Las capacidades concretas dependen del motor y la versión.

En PostgreSQL, PK y UNIQUE crean índices automáticamente; las FK del lado que referencia no reciben automáticamente uno. Cambiar el orden de una PK compuesta conserva las combinaciones únicas permitidas, pero cambia su índice de soporte.

### Ejemplo

```sql
CREATE INDEX idx_trabaja_proyecto_empleado
ON trabaja_en (cod_proyecto, id_empleado);
```

Para localizar empleados de un proyecto, el primer componente agrupa las participaciones de ese proyecto. Un índice (id_empleado,cod_proyecto) favorece la dirección inversa: proyectos de un empleado. Si esa dirección ya la cubre la PK, evaluar el índice complementario en lugar de duplicarla.

Para nombre por prefijo y correo por igualdad, `(e_mail,nombre)` permite ubicar primero el correo y después el rango de nombres. Si el correo ya es único, el índice simple existente podría localizar como máximo una fila y volver innecesario el compuesto para estas consultas.

### Errores frecuentes

No crear un índice por cada columna sin revisar los ya existentes. Muchos valores distintos no bastan para garantizar beneficio: `id > 50` puede devolver casi toda la tabla. Tampoco se debe elegir siempre «la columna más selectiva primero» ignorando igualdad, rangos y las otras consultas que compartirán el índice.

### Qué recordar

Justificar columnas, orden, consultas beneficiadas y mantenimiento. Sin estadísticas no hay una elección universalmente óptima. [Índices compuestos](https://www.postgresql.org/docs/current/indexes-multicolumn.html).

## 7. Índices según las condiciones de búsqueda

### Idea central

El índice debe corresponder a la operación que realiza el filtro. Importa tanto la columna como la expresión y el operador que se usan sobre ella.

### Cómo razonarlo

| Condición | Criterio inicial |
|---|---|
| columna = valor | B+ permite igualdad; valorar cuántas filas coinciden. |
| columna > valor / BETWEEN | B+ permite localizar y recorrer un intervalo ordenado. |
| nombre LIKE 'An%' | Hay prefijo fijo: un índice ordenado compatible puede ayudar. |
| nombre LIKE '%Maria' | No hay comienzo fijo; un B+ normal no delimita un rango de prefijo útil. |
| función(columna) = valor | Un índice sobre columna no equivale automáticamente a uno sobre la expresión. |
| varios filtros | Evaluar un compuesto o combinación de índices, sin duplicar estructuras innecesarias. |

Un filtro aprovechable como condición de acceso por índice suele llamarse **sargable**. Dejar la columna sin transformar suele facilitarlo, pero toda reescritura debe conservar el significado.

### Ejemplo

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

### Errores frecuentes

No indexar una expresión dependiente del día actual como si fuera inmutable: los datos del índice no se actualizarían solos al pasar el tiempo. No confundir «consulta frecuente» con «índice siempre rentable». SELECT * también exige recuperar muchas columnas: hallar ubicaciones no elimina el costo de devolver sus datos.

### Qué recordar

Revisar igualdad/rango/patrón, expresiones, selectividad y límites temporales. [Tipos y patrones](https://www.postgresql.org/docs/15/indexes-types.html), [expresiones](https://www.postgresql.org/docs/current/indexes-expressional.html) y [MONTHS_BETWEEN](https://docs.oracle.com/database/121/SQLRF/functions114.htm).

## 8. Árboles B+ y Bitmap

### Idea central

Las estructuras de índices resuelven necesidades diferentes. El práctico compara el árbol B+ con Bitmap: hay que elegir según las búsquedas y el modo de uso de los datos.

### Cómo razonarlo

El **árbol B+** es balanceado y tiene varios hijos por nodo. Los nodos internos guían la búsqueda; las hojas contienen entradas ordenadas y referencias a los datos, con enlaces que facilitan recorridos de rangos. No es un árbol binario. Sirve para igualdad, intervalos y acceso ordenado.

Un **índice Bitmap** asocia valores con mapas de bits que representan ubicaciones de filas. Permite combinar condiciones mediante operaciones de bits. Es un candidato típico para columnas con pocos valores distintos y cargas principalmente de consulta, con pocas modificaciones concurrentes.

| Criterio | Árbol B+ | Bitmap persistente |
|---|---|---|
| Igualdad selectiva | Habitualmente apropiado. | Puede servir; evaluar cardinalidad y carga. |
| Rangos y orden | Ventaja natural por orden de claves. | No es la primera elección del práctico para estos casos. |
| Pocos valores y combinación de filtros | Puede servir según la distribución. | Candidato típico. |
| Muchas modificaciones concurrentes | Opción general habitual. | Su mantenimiento y bloqueo pueden hacerlo inconveniente. |

### Ejemplo

Supongamos cuatro distribuidores con tipos N,I,N,I. Un bitmap conceptual para N es `1010`: señala el primero y el tercero. Otro filtro puede producir `1100`; su AND da `1000`, dejando solo el primero. Es una ilustración; las implementaciones comprimen y organizan físicamente estos mapas.

En el ejercicio 5, `tipo='N'` es el candidato natural a evaluar como Bitmap si el motor lo soporta y la carga lo favorece. Fecha e identificadores por rango se razonan inicialmente con B+.

**Diferencia de PostgreSQL:** usa la denominación B-tree para su índice ordenado habitual y no ofrece un método estándar `USING bitmap` como índice persistente. Bitmap Index Scan en un plan puede construir un mapa en ejecución a partir de un B-tree: no demuestra que se haya creado un índice Bitmap.

Hash es otra estructura: organiza búsquedas por igualdad y no proporciona orden ni rangos. No sustituye automáticamente a B+ y no es necesario añadirlo para toda igualdad.

### Errores frecuentes

«Pocos valores distintos» no basta para ordenar crear un índice: importan tamaño, combinación de filtros, proporción recuperada y mantenimiento. No confundir estructura Bitmap con estrategia de bitmap scan. No afirmar que B+ sea apropiado solo para rangos: también resuelve igualdad.

### Qué recordar

**Estructura + operador + distribución + carga de trabajo** justifican la elección. [PostgreSQL: tipos](https://www.postgresql.org/docs/15/indexes-types.html) y [bitmap scans](https://www.postgresql.org/docs/15/indexes-bitmap-scans.html); [Oracle: diseño de Bitmap](https://docs.oracle.com/en/database/oracle/oracle-database/21/tgdba/designing-and-developing-for-performance.html).

## Checklist de repaso

- [ ] Puedo leer un árbol de plan y distinguir estimado/real.
- [ ] Puedo explicar las heurísticas y las estadísticas que usa el optimizador.
- [ ] Justifico la equivalencia con claves, multiplicidades y NULL.
- [ ] Distingo JOIN, EXISTS y ausencia.
- [ ] Sé cuándo DISTINCT y COUNT cambian el resultado.
- [ ] Elijo columnas y orden de índices con una justificación.
- [ ] Analizo funciones, fechas y patrones LIKE.
- [ ] Distingo B+, Bitmap persistente y bitmap scan.

## Material y alcance

Fuentes de cátedra, dentro de Bases de Datos y fuera del baúl:

- Practicos/Practico 7 - Optimización de Consultas.docx, incluidos sus diagramas de investigación y mensajería.
- Contenido extra/Filminas y Practicos de cursada/2023/BDI_07 (Optimizacion).pdf: procesamiento, heurísticas, estadísticas, equivalencias algebraicas y EXPLAIN.

Las referencias oficiales de PostgreSQL complementan los detalles del motor; las reglas de Oracle se identifican donde las usa el enunciado. Los ejemplos numéricos del resumen son ilustrativos. Los planes comprobados y sus condiciones se documentan en la resolución.

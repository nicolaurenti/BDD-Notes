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

### Leer un árbol de operaciones

Consideremos ahora la consulta anterior con ORDER BY titulo. Este plan inventado muestra dos operaciones:

```text
Sort  (cost=12.93..12.98 rows=20 width=40)
  Sort Key: titulo
  -> Seq Scan on pelicula  (cost=0.00..12.50 rows=20 width=40)
       Filter: (genero = 'TERROR')
```

Para reconstruir el recorrido de las filas, empezar por el hijo más indentado: el Seq Scan produce las películas de terror y Sort recibe esas filas y las ordena por título. La primera línea es la raíz y entrega el resultado final. Esto describe dependencias; no significa que cada nodo deba terminar por completo antes de que otro empiece: algunas operaciones van entregando filas a medida que trabajan.

El costo del padre incluye el de sus hijos. En el ejemplo, el costo total estimado es 12.98; sumar 12.98 y 12.50 contaría parte del trabajo dos veces.

### Reconocer las operaciones habituales

| Nombre en el plan | Cómo interpretarlo |
|---|---|
| Seq Scan | Recorre la tabla y aplica los filtros correspondientes. |
| Index Scan | Usa un índice para localizar filas y accede a la tabla cuando necesita sus datos. |
| Index Only Scan | Puede obtener los valores del índice, aunque a veces necesita consultar la tabla para verificar visibilidad. |
| Bitmap Index Scan + Bitmap Heap Scan | Construye un mapa de ubicaciones desde un índice y luego busca las filas en las páginas de la tabla. No implica que el índice sea de tipo Bitmap. |
| Nested Loop | Para cada fila de una entrada, procesa la otra; esta puede usar un índice. |
| Hash Join | Construye una estructura hash con una entrada y busca coincidencias de la otra. |
| Merge Join | Combina entradas ordenadas según las claves de unión; el orden puede provenir de índices o de Sort. |
| Sort | Ordena filas. |
| Aggregate / HashAggregate / GroupAggregate | Calcula agregados; puede formar grupos según la consulta y el nodo. |
| Unique | Elimina duplicados de una entrada ordenada. DISTINCT también puede resolverse mediante HashAggregate. |

No hay una correspondencia fija entre una cláusula SQL y un único nodo. Por ejemplo, ORDER BY puede aprovechar el orden de un índice y no necesitar Sort.

Index Cond identifica una condición usada para acceder por el índice; Filter comprueba una condición sobre las filas que el nodo examina. Tener un índice no garantiza que el motor lo elija.

### Interpretar las mediciones reales

```sql
EXPLAIN ANALYZE
SELECT titulo FROM pelicula WHERE genero = 'TERROR';
```

Salida inventada:

```text
Seq Scan on pelicula  (cost=0.00..12.50 rows=20 width=40)
  (actual time=0.010..0.040 rows=18 loops=1)
  Filter: (genero = 'TERROR')
  Rows Removed by Filter: 82
Planning Time: 0.100 ms
Execution Time: 0.060 ms
```

La comparación es entre 20 filas estimadas y 18 reales. Se descartaron 82: en este ejemplo se examinaron 100. Los tiempos de actual time están en milisegundos: tiempo hasta la primera fila y hasta completar el nodo. loops indica cuántas veces se ejecutó. Si loops es mayor que 1, las filas y tiempos mostrados son promedios por ejecución; por ejemplo, rows=3 y loops=10 representa aproximadamente 30 filas emitidas en total.

Planning Time corresponde a planificar; Execution Time, a ejecutar con la instrumentación del comando. Los tiempos de nodos padres incluyen trabajo de sus hijos: tampoco deben sumarse indiscriminadamente. EXPLAIN ANALYZE agrega sobrecarga de medición; una ejecución aislada no basta para asegurar que una alternativa siempre es más rápida.

ANALYZE usado solo actualiza estadísticas; EXPLAIN ANALYZE mide la ejecución de la sentencia. Son usos distintos del término.

### Entender por qué se eligió el plan

El optimizador usa estadísticas para estimar cuántas filas producirán las operaciones. Supongamos dos escenarios ilustrativos:

- Hay un millón de películas y diez son de terror: un índice apropiado podría evitar revisar muchas filas.
- Hay cien películas y noventa son de terror: recorrer la tabla podría ser más conveniente que localizar y buscar casi todas las filas mediante el índice.

La proporción de coincidencias ayuda a razonar sobre la selectividad. No existe un porcentaje universal a partir del cual siempre convenga un índice. También influyen el tamaño, la distribución física y los costos configurados.

Si se estiman 20 filas y salen 18, la estimación es cercana. Si se estiman 20 y salen 200000, conviene investigar las estadísticas y la distribución de los datos: esa diferencia puede afectar la elección de accesos y joins. No demuestra por sí sola que falte un índice.

### Cómo explicar un plan en un ejercicio

1. Identificar qué resultado pide la consulta.
2. Localizar los accesos a las tablas y sus condiciones.
3. Seguir las filas hacia los joins, agregaciones, eliminación de duplicados y ordenamientos que existan.
4. Leer el costo global y las filas estimadas; con ANALYZE, contrastar filas y tiempos reales, considerando loops.
5. Justificar lo observado con los datos y restricciones disponibles. Sin mediciones no afirmar que una mejora propuesta es más rápida.

Para el ejemplo con Sort, una respuesta completa sería: «Se recorre pelicula secuencialmente y se filtra por TERROR. Se estiman 20 filas de salida. Esas filas se ordenan por titulo. El costo total estimado del plan es 12.98 unidades. Solo con EXPLAIN no conozco ni la duración ni la cantidad real de filas».

### Duda trabajada: ¿solo hay que aprender los dos comandos?

No. EXPLAIN y EXPLAIN ANALYZE permiten observar el plan. El aprendizaje consiste en reconocer sus operaciones, reconstruir el recorrido de las filas y distinguir estimaciones de mediciones. Las reescrituras y decisiones de índices se desarrollarán en sus puntos propios.

## 2. Reescrituras equivalentes y comparación de planes

### Qué buscamos al reescribir

Reescribir una consulta es expresar el mismo pedido con otro SQL. Buscamos facilitar un plan que haga menos trabajo, conservando el resultado. Una consulta más corta o sin subconsultas no es necesariamente más rápida: el optimizador puede transformar dos escrituras en el mismo plan.

Hay dos preguntas distintas, en este orden: ¿la nueva consulta sigue siendo correcta? ¿su ejecución mejora? Una respuesta afirmativa a la segunda no compensa una respuesta negativa a la primera.

### Qué significa conservar el resultado

La equivalencia debe valer para todos los datos permitidos por las restricciones, no solo para el ejemplo actual. Deben conservarse columnas y valores, cantidad de repeticiones de cada fila, comportamiento ante NULL y filas sin coincidencia. Si se pide orden, debe conservarse el ORDER BY; sin él no se garantiza un orden de presentación.

Las claves y restricciones son parte de la justificación. Si una reescritura necesita que id_tarea sea único, hay que comprobarlo en el esquema y explicitarlo.

### Ejemplo completo: ejercicio 2 del TP7

La consulta busca los datos de las tareas que tienen más de cinco voluntarios. Partimos de la consulta del enunciado:

```sql
SELECT *
FROM tarea
WHERE id_tarea IN (
    SELECT t.id_tarea
    FROM tarea t
    INNER JOIN voluntario v ON t.id_tarea = v.id_tarea
    GROUP BY t.id_tarea
    HAVING COUNT(v.nro_voluntario) > 5
);
```

Su razonamiento lógico es: unir las tareas con sus voluntarios, agrupar por tarea, conservar los identificadores cuyo conteo supera cinco y buscar sus datos en la tarea exterior. Esta explicación del significado no obliga al motor a ejecutar literalmente esos pasos ni a leer físicamente tarea dos veces: eso se comprueba en el plan.

Supongamos que id_tarea es la PK de tarea y nro_voluntario es la PK de voluntario. Cada voluntario unido aporta una fila al grupo de su tarea y su identificador no es NULL.

| Tarea | Voluntarios | ¿Aparece? |
|---|---:|---|
| A | 6 | Sí |
| B | 2 | No |
| C | 0 | No |

### Primera alternativa: quitar una referencia a tarea

```sql
SELECT *
FROM tarea
WHERE id_tarea IN (
    SELECT v.id_tarea
    FROM voluntario v
    GROUP BY v.id_tarea
    HAVING COUNT(v.nro_voluntario) > 5
);
```

La subconsulta puede contar directamente los voluntarios agrupados por id_tarea. Para decidir qué identificadores superan cinco no necesita atributos de tarea. La consulta exterior ya recupera los datos de las tareas existentes.

Bajo la PK declarada, el join original no multiplica voluntarios dentro de un grupo. Por eso el conteo para cada tarea existente es el mismo en ambas versiones. A sigue teniendo seis, B dos y C ninguna coincidencia que la incluya.

Incluso si hubiera voluntarios con un identificador de tarea inexistente, ese grupo no encontraría una tarea en la consulta exterior. Si hay voluntarios con id_tarea NULL, su grupo tampoco coincide con una PK exterior mediante IN. La equivalencia de este ejemplo no necesita suponer una FK, aunque el esquema puede tenerla.

La mejora propuesta consiste en eliminar una referencia lógica redundante a tarea y permitir contar directamente sobre voluntario. Solo el plan permite comprobar si efectivamente se evita trabajo que la versión original realizaba.

### Segunda alternativa: unir con los grupos ya calculados

```sql
SELECT t.*
FROM tarea t
JOIN (
    SELECT v.id_tarea
    FROM voluntario v
    GROUP BY v.id_tarea
    HAVING COUNT(v.nro_voluntario) > 5
) candidatas ON candidatas.id_tarea = t.id_tarea;
```

También expresa el mismo pedido bajo los supuestos anteriores. La tabla derivada candidatas produce como máximo una fila por id_tarea gracias al GROUP BY; su join no repite cada tarea por la cantidad de voluntarios. Se conserva t.* para devolver solamente las columnas originales.

Esto no demuestra que JOIN sea más rápido que IN. PostgreSQL podría generar planes iguales o muy similares para ambas versiones.

### Cómo comprobar la mejora

1. Justificar primero la equivalencia según las restricciones.
2. Obtener EXPLAIN de la original y las alternativas.
3. Comparar accesos a tablas, joins, agrupaciones, filas intermedias y costo total estimado de la raíz. No contar simplemente cuántas líneas tiene cada plan.
4. Medir las consultas SELECT con EXPLAIN ANALYZE sobre los mismos datos, índices y configuración, sin cambios concurrentes que alteren la comparación.
5. Contrastar filas reales y estimadas, loops y Execution Time. Repetir las mediciones alternando el orden para reducir el efecto de caché y variaciones de carga.

Si la original ya obtiene el mismo plan que la reescrita, no hay evidencia de una mejora de ejecución; la nueva escritura todavía puede resultar más clara. Si el costo estimado baja, existe una predicción de mejora; si los tiempos medidos mejoran de forma consistente en condiciones comparables, existe evidencia para esos datos y ese entorno.

No se obtuvieron planes ni tiempos locales en esta explicación. Las alternativas son propuestas justificadas, no mejoras medidas.

### Errores que invalidan una reescritura

- **Duplicados:** reemplazar un IN o EXISTS por un join con filas sin agrupar puede repetir cada fila exterior por cada coincidencia. No agregar DISTINCT automáticamente: podría eliminar repeticiones que la consulta original conservaba.
- **NULL:** NOT IN no es intercambiable sin más con NOT EXISTS. Si la subconsulta contiene NULL, un valor sin coincidencias puede producir UNKNOWN en NOT IN y quedar excluido del WHERE.
- **Filas sin coincidencia:** sustituir un LEFT JOIN por INNER JOIN puede eliminar filas. Mover condiciones entre ON y WHERE también puede cambiar un outer join.
- **Conteos y filtros:** COUNT(*) cuenta filas; COUNT(columna) omite los NULL de esa columna. WHERE filtra filas antes de agrupar y HAVING filtra grupos; no se intercambian sin justificarlo.

### Cómo verificar resultados con datos de prueba

En PostgreSQL, A EXCEPT ALL B y B EXCEPT ALL A permiten buscar diferencias conservando las cantidades de duplicados, siempre que las consultas devuelvan columnas compatibles. Ambas diferencias deben quedar vacías. Sirve como comprobación sobre esos datos, no como prueba de equivalencia para todos los estados posibles; tampoco compara el orden.

Probar tareas con cero, cinco y seis voluntarios, además de los casos con NULL que admita el esquema, ayuda a detectar errores en este ejemplo. No se ejecutaron estas comprobaciones sobre las bases del usuario.

### Qué escribir en una resolución

«Bajo la clave primaria de tarea, propongo agrupar voluntario directamente por id_tarea. Se conserva el conteo de cada tarea existente y la consulta exterior recupera sus datos. Se elimina una referencia lógica redundante a tarea. Compararía los planes y las mediciones para determinar si se reduce trabajo; la reescritura por sí sola no demuestra mayor velocidad».

Referencias: [subconsultas y NULL](https://www.postgresql.org/docs/current/functions-subquery.html), [joins, WHERE y agrupamiento](https://www.postgresql.org/docs/current/queries-table-expressions.html), [comparación de planes](https://www.postgresql.org/docs/current/using-explain.html) y [EXCEPT ALL](https://www.postgresql.org/docs/current/queries-union.html).

## 3. Joins, subconsultas y ensambles innecesarios

### Elegir según lo que pide la consulta

Un join combina filas relacionadas. Una subconsulta puede proporcionar valores, una tabla derivada o una condición de existencia. Las formas no tienen una jerarquía fija de rendimiento: primero expresar correctamente el pedido y después comparar planes.

| Necesidad | Forma que expresa directamente el pedido |
|---|---|
| Devolver datos de ambos lados o cada pareja coincidente | JOIN |
| Devolver filas de un lado que tienen alguna coincidencia | EXISTS o IN |
| Devolver filas sin coincidencias | NOT EXISTS |
| Combinar con un resultado previamente agrupado | JOIN con una tabla derivada |

Esta tabla orienta la escritura; no garantiza qué alternativa será más rápida.

### Ejemplo del ejercicio 3.a

El práctico pide los empleados investigadores que trabajan en algún proyecto:

```sql
SELECT DISTINCT e.*
FROM empleado e
JOIN trabaja_en t ON e.id_empleado = t.id_empleado
JOIN proyecto p ON p.cod_proyecto = t.cod_proyecto
WHERE e.cargo = 'investigador';
```

Para razonar, supongamos PK en empleado.id_empleado y proyecto.cod_proyecto, y PK compuesta (id_empleado, cod_proyecto) en trabaja_en. Cuando usemos la integridad referencial, supondremos que las FK se cumplen en los datos consultados.

Empleado aporta los datos de salida y el filtro por cargo. Trabaja_en demuestra la participación. Proyecto solo verifica en esta consulta que el proyecto referenciado existe: no aporta columnas al resultado ni filtros adicionales.

### Cuándo se puede quitar proyecto

Si trabaja_en.cod_proyecto tiene una FK válida hacia proyecto.cod_proyecto y es NOT NULL, toda participación encuentra exactamente un proyecto: la FK asegura existencia y la PK del destino asegura unicidad. El join no descarta ni multiplica participaciones. Puede retirarse:

```sql
SELECT DISTINCT e.*
FROM empleado e
JOIN trabaja_en t ON e.id_empleado = t.id_empleado
WHERE e.cargo = 'investigador';
```

En este esquema hipotético, cod_proyecto ya es NOT NULL por pertenecer a la PK compuesta. No alcanza con asumir una FK: hay que comprobar las restricciones del esquema del ejercicio.

La diferencia entre existencia y unicidad es importante: sin existencia garantizada el join puede excluir huérfanos; sin unicidad puede multiplicar coincidencias.

### Situaciones en las que no se puede quitar sin más

- Si cod_proyecto admite NULL, la FK puede permitir esa fila y el INNER JOIN original la excluye. En un esquema así, al quitar proyecto habría que conservar ese descarte mediante t.cod_proyecto IS NOT NULL, siempre que las restantes condiciones aseguren la equivalencia.
- Si hay referencias inexistentes, quitar el join puede incorporar empleados que antes no aparecían.
- Si se filtra por p.nombre = 'IA', proyecto decide qué participaciones cuentan: se necesita consultar esa información.
- Si se devuelve p.nombre, proyecto aporta un dato de salida.
- Si proyecto conecta dos tablas que no tienen otro enlace equivalente, funciona como puente: no es prescindible solo porque no aparezca en SELECT.

Para una FK compuesta debe verificarse la correspondencia de todas las columnas y su política de NULL. Una igualdad sobre una parte de la clave no garantiza una única coincidencia.

En la consulta original, empleado no puede retirarse directamente: se necesitan e.* y e.cargo. Trabaja_en tampoco: sin ella se perdería la condición de participar en algún proyecto.

### Evitar multiplicaciones cuando solo interesa existencia

Supongamos estas participaciones ilustrativas:

| Empleado | Cargo | Proyectos |
|---|---|---|
| Ana | investigador | 10 y 20 |
| Bruno | investigador | 10 |
| Carla | investigador | ninguno |

El join con trabaja_en produce Ana dos veces y Bruno una vez. DISTINCT devuelve Ana y Bruno una sola vez.

Podemos expresar directamente «tiene al menos una participación»:

```sql
SELECT e.*
FROM empleado e
WHERE e.cargo = 'investigador'
  AND EXISTS (
      SELECT 1
      FROM trabaja_en t
      WHERE t.id_empleado = e.id_empleado
  );
```

Bajo los supuestos de FK/NOT NULL anteriores, esta consulta conserva el resultado de la original. EXISTS conserva cada fila exterior que tiene alguna coincidencia; no genera una copia por cada participación. Como e.* incluye la PK, no hay empleados distintos con la misma fila completa: aquí no se necesita DISTINCT.

SELECT 1 no significa que se devuelva el número 1 en el resultado exterior; en este EXISTS simple solo interesa que haya una fila. La referencia a e.id_empleado hace que la subconsulta sea correlacionada: su condición depende del empleado exterior. Esto describe su significado, no obliga a ejecutar una subconsulta completa por cada empleado; PostgreSQL puede transformarla en un plan de semijoin.

Si se proyectara solamente e.nombre, dos empleados distintos llamados Ana podrían producir dos filas iguales con EXISTS; SELECT DISTINCT e.nombre produciría una sola. La equivalencia depende de la proyección y de sus duplicados.

### Otra alternativa: IN

```sql
SELECT e.*
FROM empleado e
WHERE e.cargo = 'investigador'
  AND e.id_empleado IN (
      SELECT t.id_empleado FROM trabaja_en t
  );
```

En este uso positivo de IN dentro de WHERE, se seleccionan los empleados cuyo identificador coincide con alguna participación. Los identificadores repetidos de la subconsulta no repiten la fila exterior. Puede resultar equivalente a EXISTS y producir un plan parecido; no hay que convertir siempre IN en JOIN.

### Buscar ausencia y tratar NULL

Para investigadores sin participaciones:

```sql
SELECT e.*
FROM empleado e
WHERE e.cargo = 'investigador'
  AND NOT EXISTS (
      SELECT 1
      FROM trabaja_en t
      WHERE t.id_empleado = e.id_empleado
  );
```

NOT EXISTS es verdadero cuando no hay coincidencias. NOT IN requiere más cuidado: si su subconsulta produce NULL y no se encuentra una igualdad, puede dar UNKNOWN y excluir la fila. Una PK no nullable elimina ese problema concreto, pero no debe suponerse para cualquier consulta.

LEFT JOIN seguido de WHERE t.id_empleado IS NULL también puede expresar ausencia si se comprueba una columna del lado derecho que no puede ser NULL en una coincidencia real. Comprobar una columna nullable ajena a la unión podría confundir una coincidencia con valor NULL con una falta de coincidencia.

### Qué buscar en el plan y cómo justificar

Tras una reescritura, comprobar si desaparece el acceso a proyecto y si se evita producir filas para luego eliminar duplicados. Un EXISTS o IN puede verse como Hash Semi Join, Nested Loop Semi Join u otra estrategia; no asumir un nodo obligatorio. NOT EXISTS puede resolverse con un anti join. Mirar filas reales y loops para verificar el trabajo realizado.

La conclusión correcta distingue la equivalencia demostrada de una mejora de ejecución que hay que medir. Cambiar comas y condiciones en WHERE por JOIN ... ON hace más clara una unión interna, pero no garantiza otro plan ni fuerza por sí solo el orden físico de ejecución. Tampoco se pueden mover condiciones de outer joins libremente como si fueran inner joins.

### Método para detectar ensambles innecesarios

Para cada tabla, identificar qué aporta: columnas, filtros, existencia, multiplicidad o conexión con otras tablas. Si parece prescindible, demostrar que quitarla no incorpora filas, no elimina filas y no cambia repeticiones. Usar PK/UNIQUE, FK, NOT NULL y condiciones completas del join. Después comparar planes y mediciones.

Referencias: [joins y multiplicidad](https://www.postgresql.org/docs/current/queries-table-expressions.html), [EXISTS, IN y NOT IN](https://www.postgresql.org/docs/current/functions-subquery.html), [restricciones](https://www.postgresql.org/docs/current/ddl-constraints.html) y [orden de joins](https://www.postgresql.org/docs/current/explicit-joins.html).

## Temas pendientes

4. DISTINCT y eliminación de duplicados.
5. GROUP BY y HAVING.
6. Índices simples, compuestos y orden de columnas.
7. Índices ante igualdad, rangos, fechas, funciones y LIKE.
8. Árboles B+ y Bitmap.

## Material

Enunciado: Practicos/Practico 7 - Optimización de Consultas.docx, dentro de Bases de Datos, fuera del baúl. El práctico propone analizar planes y justificar índices.

Referencias técnicas: [Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html), [comando EXPLAIN](https://www.postgresql.org/docs/current/sql-explain.html) y [estadísticas del optimizador](https://www.postgresql.org/docs/current/planner-stats.html).

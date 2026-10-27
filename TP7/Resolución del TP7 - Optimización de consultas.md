---
tags:
  - tp7
---

# Resolución del TP7 - Optimización de consultas

Resolución del enunciado principal de Practicos. Cada ejercicio sigue: **pedido → razonamiento → SQL → interpretación del plan → conclusión**. Se distinguen soluciones equivalentes, propuestas de índices y comprobaciones realizadas.

## Condiciones de lectura y validación

Se utilizó PostgreSQL 16.14 en una instancia temporal aislada, por socket exclusivo y sin escucha TCP. Antes de cada lote se verificaron base postgres, esquema y directorio temporal. No se modificaron las bases habituales, no se borró ninguna base y los índices propuestos no se crearon en las conexiones de estudio.

- Ejercicios 1–2 y 5: copias temporales de los scripts locales de Películas y Voluntarios; datos sintéticos de práctica. Para 1.a se agregó solo en la copia una película TERROR con un renglón, para comprobar un resultado no vacío.
- Ejercicio 3: diagrama y seis empleados, tres proyectos y once participaciones exactos del enunciado. Se crearon dos esquemas separados para comparar los dos órdenes de PK.
- Ejercicio 4: diagrama original, con datos sintéticos y casos de cero, dos y tres grupos exclusivos. También se construyó un contraejemplo de inconsistencia entre discriminante y subtipo.

Los árboles incluidos son **transcripciones abreviadas de planes reales** de esa instancia: se conservaron operaciones, condiciones relevantes y cantidades; no representan las bases habituales. Los costos son estimaciones del entorno temporal; los tiempos de tablas diminutas no demuestran mejoras generales.

Para observar un plan en una conexión de estudio, primero comprobar:

```sql
SELECT current_database() AS base,
       current_schema() AS esquema,
       current_setting('search_path') AS ruta;
```

Los ejemplos CREATE INDEX son propuestas documentadas, no instrucciones para ejecutarlas todas. No contienen cambios de PK ni recargas de datos. Si se desean medir, usar una copia aislada y revisar los índices ya existentes.

## Ejercicio 1 — Analizar planes con EXPLAIN

### 1.a. Entregas con películas de terror

**Pedido.** Mostrar título y número de entrega para cada película de terror incluida en una entrega.

**Razonamiento.** Filtrar pelicula por género y unir sus códigos con renglon_entrega. El SQL original une además entrega para obtener el número que ya está en renglon_entrega.

**SQL original, con comillas normalizadas y JOIN explícito:**

```sql
EXPLAIN
SELECT p.titulo, e.nro_entrega
FROM pelicula p
JOIN renglon_entrega re
  ON re.codigo_pelicula = p.codigo_pelicula
JOIN entrega e ON e.nro_entrega = re.nro_entrega
WHERE p.genero = 'TERROR';
```

La consulta no devuelve una sola fila por entrega: si una entrega tiene dos películas de terror, devuelve dos filas. Es coherente con la proyección del enunciado, que incluye el título. No agregar DISTINCT sobre títulos para forzar una fila por entrega: dos películas distintas pueden compartir título.

**Interpretación del plan observado:**

```text
Hash Join [Inner]  (estimadas=3, reales=1, loops=1)
  Hash Cond: (e.nro_entrega = re.nro_entrega)
  Seq Scan on entrega  (estimadas=12, reales=12, loops=1)
  Hash  (estimadas=3, reales=1, loops=1)
    Hash Join [Inner]  (estimadas=3, reales=1, loops=1)
      Hash Cond: (re.codigo_pelicula = p.codigo_pelicula)
      Seq Scan on renglon_entrega  (estimadas=36, reales=36, loops=1)
      Hash  (estimadas=1, reales=1, loops=1)
        Seq Scan on pelicula  (estimadas=1, reales=1, loops=1)
          Filter: ((genero)::text = 'TERROR'::text)
```

El filtro se aplica al leer pelicula, antes de que sus filas participen en las uniones. Los Hash Join buscan coincidencias por las claves; la elección de recorridos secuenciales es razonable con tablas pequeñas. La instancia temporal devolvió (Terror TP7,1), agregado sintético exclusivo de la comprobación.

El nodo de pelicula redujo trece películas a una. El motor no ejecutó materialmente un producto cartesiano completo de las tres tablas: transformó las igualdades en uniones.

**Alternativa equivalente bajo las restricciones del script local:**

```sql
SELECT p.titulo, re.nro_entrega
FROM pelicula p
JOIN renglon_entrega re
  ON re.codigo_pelicula = p.codigo_pelicula
WHERE p.genero = 'TERROR';
```

La FK de renglon_entrega hacia entrega asegura existencia; nro_entrega pertenece a la PK compuesta de renglon_entrega y no admite NULL; entrega tiene PK nro_entrega. Por ello el join retirado encontraba exactamente una fila y no añadía filtros ni columnas distintas. Si esas restricciones no se cumplen, hay que revisar la equivalencia.

**Índices candidatos, si volumen y selectividad lo justifican:** pelicula(genero) y renglon_entrega(codigo_pelicula,nro_entrega). La PK local del renglón comienza por nro_entrega; la alternativa permite buscar en la dirección película → renglones. La PK de entrega ya cubre búsquedas por nro_entrega. No crear automáticamente ambos candidatos en una tabla pequeña.

**Conclusión.** La heurística visible es reducir pelicula por género antes de unir; la elección física observada es por costo. Puede simplificarse retirando entrega bajo las restricciones verificadas.

### 1.b. Voluntarios con históricos que cumplen horas y fecha

**Discrepancia del enunciado.** El texto pide finalizados antes de 01/01/2023, pero su SQL compara contra 24/07/1998. Son pedidos distintos: se conservan ambas versiones explícitamente.

**SQL que reproduce la consulta dada:**

```sql
EXPLAIN
SELECT v.nombre, v.apellido, v.e_mail, v.telefono
FROM voluntario v
WHERE v.nro_voluntario IN (
    SELECT h.nro_voluntario
    FROM historico h
    WHERE h.fecha_fin < DATE '1998-07-24'
      AND h.id_tarea IN (
          SELECT t.id_tarea
          FROM tarea t
          WHERE t.max_horas - t.min_horas <= 5000
      )
);
```

El criterio de horas es la diferencia max_horas − min_horas de tarea: no las horas aportadas por el voluntario ni la duración entre fecha_inicio y fecha_fin. Se selecciona a quien tiene al menos un histórico que cumple ambas condiciones; no a quien las cumple en todos sus históricos.

**Interpretación del plan observado:**

```text
Hash Join [Semi]  (estimadas=1, reales=2, loops=1)
  Hash Cond: (v.nro_voluntario = h.nro_voluntario)
  Seq Scan on voluntario  (estimadas=111, reales=111, loops=1)
  Hash  (estimadas=1, reales=2, loops=1)
    Hash Join [Inner]  (estimadas=1, reales=2, loops=1)
      Hash Cond: ((t.id_tarea)::text = (h.id_tarea)::text)
      Seq Scan on tarea  (estimadas=7, reales=10, loops=1)
        Filter: ((max_horas - min_horas) <= '5000'::numeric)
      Hash  (estimadas=2, reales=3, loops=1)
        Seq Scan on historico  (estimadas=2, reales=3, loops=1)
          Filter: (fecha_fin < '1998-07-24'::date)
```

Se filtran tarea por amplitud de horas e historico por fecha. Los resultados se relacionan con los voluntarios. Aunque el SQL tenga dos IN, el motor eligió joins hash; no realizó necesariamente dos subconsultas completas independientes por voluntario. Una estrategia de pertenencia también puede implementarse con joins cuyas entradas sean únicas o con semijoins; no hay un único nombre de nodo obligatorio.

**Escritura equivalente y fácil de explicar, conservando la fecha del SQL:**

```sql
SELECT v.nombre, v.apellido, v.e_mail, v.telefono
FROM voluntario v
WHERE EXISTS (
    SELECT 1
    FROM historico h
    JOIN tarea t ON t.id_tarea = h.id_tarea
    WHERE h.nro_voluntario = v.nro_voluntario
      AND h.fecha_fin < DATE '1998-07-24'
      AND t.max_horas - t.min_horas <= 5000
);
```

Para responder al **texto** del enunciado, cambiar únicamente el límite a `DATE '2023-01-01'`. EXISTS mantiene una fila por voluntario aun si tiene varios históricos válidos. No añadir DISTINCT sobre los cuatro datos de contacto: podría fusionar voluntarios distintos con contactos iguales.

**Resultado de control en la copia.** Con 1998 se obtuvieron Neena Kochhar y Jennifer Whalen. Con 2023 se obtuvieron seis: las anteriores, Den Raphaely, Payam Kaufling, Michael Hartstein y Lucía Practica Alvarez. Este resultado depende de la copia de datos, no es una respuesta universal del enunciado.

**Índices a evaluar.** Para búsquedas correlacionadas: historico(nro_voluntario,fecha_fin) o (nro_voluntario,fecha_fin,id_tarea). Para empezar por una fecha muy restrictiva: otro orden podría resultar mejor. tarea ya tiene índice por PK; un índice de expresión sobre (max_horas-min_horas) solo merece evaluarse si tamaño, frecuencia y selectividad lo justifican. Las dos primeras columnas del compuesto no convierten el tercer componente en otro rango independiente.

**Conclusión.** Aplicar los filtros locales antes de relacionar, preservar la semántica de existencia y aclarar qué fecha se está resolviendo.

## Ejercicio 2 — Mejorar el conteo de voluntarios por tarea

### Pedido y razonamiento

La consulta original obtiene las tareas con más de cinco voluntarios actuales. Dentro del IN une tarea con voluntario, agrupa y después consulta tarea otra vez. Para contar por id_tarea no necesita los atributos de tarea dentro del grupo.

### SQL original para comparar

```sql
EXPLAIN ANALYZE
SELECT *
FROM tarea
WHERE id_tarea IN (
    SELECT t.id_tarea
    FROM tarea t
    JOIN voluntario v ON t.id_tarea = v.id_tarea
    GROUP BY t.id_tarea
    HAVING COUNT(v.nro_voluntario) > 5
);
```

### SQL propuesto

```sql
EXPLAIN ANALYZE
SELECT t.*
FROM tarea t
WHERE t.id_tarea IN (
    SELECT v.id_tarea
    FROM voluntario v
    GROUP BY v.id_tarea
    HAVING COUNT(*) > 5
);
```

La PK de tarea garantiza que el join original no multiplicaba voluntarios; nro_voluntario es NOT NULL y permite reemplazar COUNT(v.nro_voluntario) por COUNT(*). Para cada tarea existente se conserva la cantidad. Las tareas con cero o cinco no aparecen; seis es el primer conteo aceptado.

Una segunda forma equivalente es:

```sql
SELECT t.*
FROM tarea t
JOIN (
    SELECT v.id_tarea
    FROM voluntario v
    GROUP BY v.id_tarea
    HAVING COUNT(*) > 5
) c ON c.id_tarea = t.id_tarea;
```

El agrupamiento produce como máximo una fila por id_tarea: este join no introduce una fila exterior por voluntario.

### Planes e interpretación

**Original:**

```text
Hash Join [Inner]  (estimadas=7, reales=3, loops=1)
  Hash Cond: ((tarea.id_tarea)::text = (t.id_tarea)::text)
  Seq Scan on tarea  (estimadas=20, reales=20, loops=1)
  Hash  (estimadas=7, reales=3, loops=1)
    HashAggregate  (estimadas=7, reales=3, loops=1)
      Filter: (count(v.nro_voluntario) > 5)
      Group Key: ['t.id_tarea']
      Hash Join [Inner]  (estimadas=111, reales=111, loops=1)
        Hash Cond: ((v.id_tarea)::text = (t.id_tarea)::text)
        Seq Scan on voluntario  (estimadas=111, reales=111, loops=1)
        Hash  (estimadas=20, reales=20, loops=1)
          Seq Scan on tarea  (estimadas=20, reales=20, loops=1)
```

**Propuesta:**

```text
Hash Join [Inner]  (estimadas=6, reales=3, loops=1)
  Hash Cond: ((tarea.id_tarea)::text = (v.id_tarea)::text)
  Seq Scan on tarea  (estimadas=20, reales=20, loops=1)
  Hash  (estimadas=6, reales=3, loops=1)
    HashAggregate  (estimadas=6, reales=3, loops=1)
      Filter: (count(*) > 5)
      Group Key: ['v.id_tarea']
      Seq Scan on voluntario  (estimadas=111, reales=111, loops=1)
```

En la copia con 111 voluntarios y veinte tareas, desaparece el join interno con tarea. El costo total estimado pasó de 7.07 a 5.24 unidades. Ambas devolvieron tres tareas: SA_REP, ST_CLERK y SH_CLERK. Se verificó igualdad de resultados con diferencias en ambos sentidos mediante EXCEPT ALL.

### Conclusión

Sí, hay una simplificación justificada y en esta copia se observó menos trabajo en el plan. No se afirma una mejora universal de tiempo: las tablas son pequeñas. Un índice por voluntario(id_tarea) podría ayudar en otras estrategias, pero no evita por sí mismo contar las filas relevantes de todos los grupos.

## Ejercicio 3 — Empleados y proyectos

### Esquema y datos utilizados

El diagrama muestra empleado con PK id_empleado; proyecto con PK cod_proyecto; trabaja_en con ambas columnas como PK/FK y cant_horas. Las FK están en trabaja_en hacia las otras dos tablas. Los datos usados en la comprobación son exactamente los del enunciado.

### 3.a. Investigadores que trabajan en algún proyecto

**SQL original, normalizado:**

```sql
EXPLAIN ANALYZE
SELECT DISTINCT e.*
FROM empleado e
JOIN trabaja_en t ON t.id_empleado = e.id_empleado
JOIN proyecto p ON p.cod_proyecto = t.cod_proyecto
WHERE e.cargo = 'investigador';
```

**Resultado:**

| id_empleado | nombre | apellido | cargo |
|---:|---|---|---|
| 1 | Juan | Perez | investigador |
| 2 | Rosa | Gomez | investigador |
| 5 | Alejandro | Perez | investigador |
| 6 | Sebastian | Solano | investigador |

El orden de esta tabla es explicativo: sin ORDER BY, el SQL no lo garantiza.

**a.1 — ¿Es necesario el ensamble con empleado?** Sí para esta consulta: aporta e.* y permite filtrar cargo. No se puede saber qué participantes son investigadores solo con trabaja_en.

**a.2 — ¿Es necesario el ensamble con proyecto?** No bajo el diagrama y las FK cumplidas. cod_proyecto en trabaja_en es parte de PK, por tanto NOT NULL, y referencia una PK de proyecto: cada participación encuentra exactamente un proyecto. No se usa ningún otro atributo de proyecto. Si se filtrara por nombre o las referencias no estuvieran garantizadas, no valdría retirarlo sin más.

**a.3 — ¿Qué implica DISTINCT?** El join produce siete filas de investigadores: Juan dos, Rosa una, Alejandro una y Sebastian tres. DISTINCT las reduce a cuatro empleados. Que id_empleado sea PK no evita que el join repita e.*. Eliminar duplicados puede requerir ordenamiento/Unique o agrupamiento hash.

**Reescritura por existencia:**

```sql
SELECT e.*
FROM empleado e
WHERE e.cargo = 'investigador'
  AND EXISTS (
      SELECT 1 FROM trabaja_en t
      WHERE t.id_empleado = e.id_empleado
  );
```

No hace falta DISTINCT porque se conserva cada fila exterior una vez. Esto depende de devolver e.*: si se devuelve solo apellido, empleados diferentes pueden compartirlo.

**Plan original observado:**

```text
HashAggregate  (estimadas=4, reales=4, loops=1)
  Group Key: ['e.id_empleado', 'e.nombre', 'e.apellido']
  Hash Join [Inner]  (estimadas=7, reales=7, loops=1)
    Hash Cond: (t.cod_proyecto = p.cod_proyecto)
    Hash Join [Inner]  (estimadas=7, reales=7, loops=1)
      Hash Cond: (t.id_empleado = e.id_empleado)
      Seq Scan on trabaja_en  (estimadas=11, reales=11, loops=1)
      Hash  (estimadas=4, reales=4, loops=1)
        Seq Scan on empleado  (estimadas=4, reales=4, loops=1)
          Filter: ((cargo)::text = 'investigador'::text)
    Hash  (estimadas=3, reales=3, loops=1)
      Seq Scan on proyecto  (estimadas=3, reales=3, loops=1)
```

**Plan de EXISTS observado:**

```text
Hash Join [Semi]  (estimadas=4, reales=4, loops=1)
  Hash Cond: (e.id_empleado = t.id_empleado)
  Seq Scan on empleado  (estimadas=4, reales=4, loops=1)
    Filter: ((cargo)::text = 'investigador'::text)
  Hash  (estimadas=11, reales=11, loops=1)
    Seq Scan on trabaja_en  (estimadas=11, reales=11, loops=1)
```

La original une y agrupa para quitar repeticiones. En la propuesta desaparecen proyecto y el agrupamiento de salida; el Hash Join es de tipo Semi. El costo estimado pasa de 3.49 a 2.38. Son planes observados para estas filas, no una regla sobre qué algoritmo debe elegirse siempre.

**Índice candidato.** La PK (id_empleado,cod_proyecto) ya favorece buscar participaciones de un empleado. No añadir otro índice idéntico por id_empleado sin justificarlo.

### 3.b. Proyectos de IA con empleados trabajando

**SQL original:**

```sql
EXPLAIN ANALYZE
SELECT DISTINCT p.*
FROM trabaja_en t
JOIN proyecto p ON p.cod_proyecto = t.cod_proyecto
WHERE p.nombre = 'IA';
```

El proyecto IA es el código 2. Tiene tres empleados —1,3,6—: el join produce tres filas del mismo proyecto y DISTINCT deja una.

| cod_proyecto | nombre | fecha_desde | fecha_hasta |
|---:|---|---|---|
| 2 | IA | 2017-01-01 | 2020-01-01 |

**b.1 — PK (id_empleado,cod_proyecto).** El índice B-tree de soporte ordena primero por empleado. Es apropiado para buscar proyectos de un empleado; no reúne en un único prefijo todas las participaciones del proyecto 2. Se puede evaluar un índice adicional (cod_proyecto,id_empleado), si esta dirección es frecuente y el volumen lo justifica. No afirmar que la segunda columna sea absolutamente inutilizable: importan motor, versión y costo.

**b.2 — PK (cod_proyecto,id_empleado).** Ordena primero por proyecto, facilitando localizar las participaciones del proyecto 2 y comprobar existencia. La unicidad de las parejas es la misma; cambia el acceso físico. No cambiar la PK en la base habitual solo para ensayar: comparar esquemas separados o usar un índice complementario en una copia.

**Lo que ocurrió con los datos del enunciado:** ambos esquemas eligieron el mismo árbol y el mismo costo estimado 2.27. Con once participaciones, leer la tabla puede costar menos que usar un índice. Cambiar el orden de PK no obliga a que cambie el plan.

```text
HashAggregate  (estimadas=1, reales=1, loops=1)
  Group Key: ['p.cod_proyecto', 'p.fecha_desde', 'p.fecha_hasta']
  Hash Join [Inner]  (estimadas=4, reales=3, loops=1)
    Hash Cond: (t.cod_proyecto = p.cod_proyecto)
    Seq Scan on trabaja_en  (estimadas=11, reales=11, loops=1)
    Hash  (estimadas=1, reales=1, loops=1)
      Seq Scan on proyecto  (estimadas=1, reales=1, loops=1)
        Filter: ((nombre)::text = 'IA'::text)
```

**Alternativa sin generar copias del proyecto:**

```sql
SELECT p.*
FROM proyecto p
WHERE p.nombre = 'IA'
  AND EXISTS (
      SELECT 1 FROM trabaja_en t
      WHERE t.cod_proyecto = p.cod_proyecto
  );
```

Si hubiera varios proyectos llamados IA, devuelve cada proyecto con participantes una vez; no hay que suponer UNIQUE(nombre). Puede evaluarse proyecto(nombre) si hay muchos proyectos y ese filtro es restrictivo.

**Conclusión.** El orden de una clave compuesta importa para la dirección de búsqueda, pero la elección física depende del tamaño y de los costos. DISTINCT es necesario en el join original para este resultado y se evita expresando existencia.

## Ejercicio 4 — Usuarios en más de dos grupos exclusivos

### Diagrama y supuestos

- Usuario: PK (tipo_usuario,cod_usuario).
- Integra: PK (tipo_usuario,cod_usuario,nro_grupo), con FK hacia usuario y grupo.
- Grupo: PK nro_grupo y atributo tipo_gr.
- Gr_exclusivo y gr_comun: cada uno con PK/FK nro_grupo hacia grupo.

Las PK evitan dos pertenencias del mismo usuario al mismo grupo y dos filas del mismo grupo dentro de gr_exclusivo. Las FK por sí solas **no garantizan** que pertenecer a gr_exclusivo equivalga a tipo_gr='E': esa coherencia del subtipo necesita una restricción o un supuesto adicional.

### 4.a. ¿La consulta responde a lo pedido?

Bajo un modelo coherente donde gr_exclusivo identifica los grupos exclusivos y el discriminante coincide, sí. Une usuarios con pertenencias, selecciona los grupos exclusivos, agrupa por la clave completa y sus datos y exige COUNT(*)>2, es decir al menos tres.

Cada fila del join cuenta un grupo diferente por usuario gracias a las PK. No necesita COUNT(DISTINCT nro_grupo) para este esquema. Si se ignora tipo_usuario se pueden mezclar usuarios distintos con el mismo cod_usuario.

Si el discriminante y el subtipo se contradicen, la original exige ambas condiciones; dejaría fuera grupos que solo cumplan una. La respuesta sobre corrección debe declarar qué define «exclusivo» y la coherencia asumida.

### 4.b. ¿Está optimizada? Propuesta bajo coherencia del modelo

Puede simplificarse: el join con grupo y su filtro son redundantes si gr_exclusivo garantiza justamente ese tipo. Además, conviene proponer agrupar las claves antes de traer nombres:

```sql
SELECT u.tipo_usuario, u.cod_usuario, u.apellido, u.nombre
FROM usuario u
JOIN (
    SELECT i.tipo_usuario, i.cod_usuario
    FROM integra i
    JOIN gr_exclusivo ge ON ge.nro_grupo = i.nro_grupo
    GROUP BY i.tipo_usuario, i.cod_usuario
    HAVING COUNT(*) > 2
) c ON c.tipo_usuario = u.tipo_usuario
   AND c.cod_usuario = u.cod_usuario;
```

Agrupar por dos columnas evita transportar apellido y nombre en ese paso. La tabla derivada produce una fila por usuario aceptado. No se elimina usuario porque aporta los datos descriptivos.

**Alternativa que preserva literalmente las dos condiciones, aun sin coherencia del discriminante:**

```sql
SELECT u.tipo_usuario, u.cod_usuario, u.apellido, u.nombre
FROM usuario u
JOIN (
    SELECT i.tipo_usuario, i.cod_usuario
    FROM integra i
    JOIN grupo g ON g.nro_grupo = i.nro_grupo
    JOIN gr_exclusivo ge ON ge.nro_grupo = g.nro_grupo
    WHERE g.tipo_gr = 'E'
    GROUP BY i.tipo_usuario, i.cod_usuario
    HAVING COUNT(*) > 2
) c ON c.tipo_usuario = u.tipo_usuario
   AND c.cod_usuario = u.cod_usuario;
```

Esta conserva la semántica original y cambia cuándo se recuperan los datos de usuario. «Propuesta optimizada» no equivale a óptimo demostrado: comparar el plan y medir.

### 4.c. Otra escritura equivalente y planes

Con coherencia del modelo, también puede usarse una condición correlacionada sobre los grupos:

```sql
SELECT u.tipo_usuario, u.cod_usuario, u.apellido, u.nombre
FROM usuario u
WHERE EXISTS (
    SELECT 1
    FROM integra i
    JOIN gr_exclusivo ge ON ge.nro_grupo = i.nro_grupo
    WHERE i.tipo_usuario = u.tipo_usuario
      AND i.cod_usuario = u.cod_usuario
    GROUP BY i.tipo_usuario, i.cod_usuario
    HAVING COUNT(*) > 2
);
```

No basta un EXISTS sin GROUP BY/HAVING: eso pediría un grupo o más, no más de dos. Para conservar ambas condiciones sin coherencia del modelo, incluir también grupo y su filtro dentro de esta subconsulta.

**Plan de la original con datos coherentes:**

```text
HashAggregate  (estimadas=1, reales=1, loops=1)
  Filter: (count(*) > 2)
  Group Key: ['u.tipo_usuario', 'u.cod_usuario']
  Hash Join [Inner]  (estimadas=4, reales=5, loops=1)
    Hash Cond: ((i.tipo_usuario = u.tipo_usuario) AND (i.cod_usuario = u.cod_usuario))
    Hash Join [Inner]  (estimadas=4, reales=5, loops=1)
      Hash Cond: (i.nro_grupo = ge.nro_grupo)
      Hash Join [Inner]  (estimadas=5, reales=5, loops=1)
        Hash Cond: (i.nro_grupo = g.nro_grupo)
        Seq Scan on integra  (estimadas=7, reales=7, loops=1)
        Hash  (estimadas=3, reales=3, loops=1)
          Seq Scan on grupo  (estimadas=3, reales=3, loops=1)
            Filter: (tipo_gr = 'E'::bpchar)
      Hash  (estimadas=3, reales=3, loops=1)
        Seq Scan on gr_exclusivo  (estimadas=3, reales=3, loops=1)
    Hash  (estimadas=4, reales=4, loops=1)
      Seq Scan on usuario  (estimadas=4, reales=4, loops=1)
```

**Plan de la primera propuesta:**

```text
Nested Loop [Inner]  (estimadas=1, reales=1, loops=1)
  Join Filter: ((u.tipo_usuario = i.tipo_usuario) AND (u.cod_usuario = i.cod_usuario))
  HashAggregate  (estimadas=1, reales=1, loops=1)
    Filter: (count(*) > 2)
    Group Key: ['i.tipo_usuario', 'i.cod_usuario']
    Hash Join [Inner]  (estimadas=5, reales=5, loops=1)
      Hash Cond: (i.nro_grupo = ge.nro_grupo)
      Seq Scan on integra  (estimadas=7, reales=7, loops=1)
      Hash  (estimadas=3, reales=3, loops=1)
        Seq Scan on gr_exclusivo  (estimadas=3, reales=3, loops=1)
  Seq Scan on usuario  (estimadas=4, reales=1, loops=1)
```

En la copia, Ana pertenece a tres grupos exclusivos y uno común; Beto a dos exclusivos; Caro solo a uno común; Dani a ninguno. Se obtiene únicamente Ana. La original transporta las pertenencias hasta usuario y luego agrupa; la propuesta cuenta primero y une el usuario aceptado al final. Sus costos estimados fueron 4.50 y 3.35 respectivamente. No trasladar esos costos a otra base.

También se comprobó un contraejemplo: si se agrega a gr_exclusivo un grupo cuyo tipo_gr es C y Beto pasa a tener tres filas de ese subtipo, la propuesta sin grupo incluye a Beto y la original lo excluye. La alternativa que conserva el filtro siguió siendo equivalente. Esto demuestra por qué el supuesto de coherencia es necesario.

**Índices candidatos.** Las PK de usuario y gr_exclusivo ya soportan sus búsquedas. La PK de integra empieza por usuario y ayuda a la búsqueda correlacionada. Un índice inverso integra(nro_grupo,tipo_usuario,cod_usuario) puede servir si se comienza desde los grupos exclusivos. El costo de leer y agrupar gran parte de integra no desaparece automáticamente por agregarlo.

**Conclusión.** Correcta bajo modelo coherente; puede expresarse con menos referencias y agrupando antes los datos descriptivos. Conservar la clave completa y explicitar la coherencia del subtipo.

## Ejercicio 5 — Elegir y compartir índices

### Método de decisión

Las consultas son frecuentes, pero no se dan tamaños, distribuciones ni tasas de actualización. Por eso se justifican candidatos y condiciones de utilidad, no se prometen mejoras universales. Revisar índices por PK/UNIQUE antes de agregar otros. El SQL local de Películas tiene PK de distribuidor(id_distribuidor) y empleado(id_empleado); no declara UNIQUE en empleado.e_mail. No trasladar aquí el índice único de correo que existe en Voluntarios.

### b.1. MONTHS_BETWEEN(SYSDATE,fecha_entrega)>100

**Consulta original de Oracle:**

```sql
SELECT *
FROM entrega
WHERE MONTHS_BETWEEN(SYSDATE, fecha_entrega) > 100;
```

**Análisis.** Se aplica una función a fecha_entrega y su resultado depende del momento actual. Un B+ ordinario sobre fecha_entrega no convierte automáticamente este predicado en un intervalo de acceso. Tampoco corresponde indexar directamente esa expresión móvil como si su valor permaneciera constante entre modificaciones de filas.

**Propuesta.** Si el pedido se define como «fecha anterior al límite calendario de cien meses», comparar directamente la columna con el límite. En PostgreSQL, esa interpretación se escribe:

```sql
SELECT *
FROM entrega
WHERE fecha_entrega < CURRENT_DATE - INTERVAL '100 months';
```

Puede evaluarse:

```sql
CREATE INDEX idx_entrega_fecha_tp7 ON entrega (fecha_entrega);
```

**Límite importante.** No se presenta la reescritura como equivalencia exacta de MONTHS_BETWEEN: Oracle utiliza reglas específicas para mismo día, fin de mes, horas y fracciones de 31 días. Si debe preservarse exactamente el original, mantener su predicado y estudiar su semántica antes de proponer una transformación. SYSDATE tampoco equivale sin más a CURRENT_DATE de PostgreSQL en todos los aspectos.

**Conclusión.** B+ sobre fecha es candidato para la variante por intervalo directo. Para el SQL original no justificar un índice simple afirmando que elimina el cálculo de la función. Si casi todas las entregas superan esa antigüedad, un recorrido secuencial puede seguir siendo preferible.

### b.2. id_distribuidor>50

```sql
SELECT * FROM distribuidor WHERE id_distribuidor > 50;
```

El índice B-tree de la PK ya permite una búsqueda por rango. **No crear otro índice idéntico.** Aunque el identificador tenga muchos valores distintos, el umbral podría incluir casi toda la tabla: depende de la distribución. En ese caso puede convenir recorrerla.

### b.3. tipo='N'

```sql
SELECT * FROM distribuidor WHERE tipo = 'N';
```

Tipo distingue pocos valores, como nacional/internacional. Un B+ por tipo puede ayudar si N corresponde a una fracción pequeña, pero no necesariamente si corresponde a gran parte de las filas. Es el candidato del práctico para **Bitmap**, si el motor lo soporta y hay muchos datos, frecuentes consultas y pocas modificaciones concurrentes.

En PostgreSQL no se escribe CREATE BITMAP INDEX ni USING bitmap. Un posible B-tree se documenta así, sujeto a justificación:

```sql
CREATE INDEX idx_distribuidor_tipo_tp7 ON distribuidor (tipo);
```

Un Bitmap Heap Scan que use este índice sería una estrategia de ejecución sobre un B-tree, no un índice Bitmap persistente.

### b.4. Igualdad por nombre y correo

```sql
SELECT * FROM empleado
WHERE nombre = 'Juan Lucas'
  AND e_mail = 'Pla@gmail.com';
```

Ambos predicados son igualdades. Un índice compuesto puede utilizar ambos; para compartirlo con b.5 y b.6 propongo empezar por correo:

```sql
CREATE INDEX idx_empleado_mail_nombre_tp7
ON empleado (e_mail, nombre);
```

Si el correo identifica muy pocas filas, un índice simple por e_mail puede ser una alternativa más pequeña y suficiente, aplicando nombre como filtro residual. Si ya existe UNIQUE(e_mail), revisar su índice antes de crear un compuesto: no hay evidencia de que se necesite otro para estas consultas.

No declarar UNIQUE solo para acelerar: agrega una restricción de negocio que la consigna no pidió.

### b.5. Prefijo de nombre y correo por igualdad

```sql
SELECT * FROM empleado
WHERE nombre LIKE 'An%'
  AND e_mail = 'Marise@gmail.com';
```

La igualdad de correo delimita la primera parte del índice (e_mail,nombre); dentro de ese correo, el prefijo An permite considerar un rango de nombres compatible con la intercalación. Es un orden razonable para compartir el índice con b.4 y b.6. Un índice (nombre,e_mail) empieza por un rango de nombres y puede no acotar el tramo de búsqueda de igual manera con el correo posterior.

**Detalle PostgreSQL.** Según la intercalación, LIKE por prefijo puede requerir una clase de operadores. Para las columnas varchar del script local:

```sql
-- Alternativa al compuesto anterior, no propuesta de crear ambos.
CREATE INDEX idx_empleado_mail_nombre_pat_tp7
ON empleado (e_mail, nombre varchar_pattern_ops);
```

La clase pattern_ops admite también igualdad; evaluar si otras consultas necesitan el orden lingüístico normal del nombre. No prometer una búsqueda por prefijo con cualquier índice y configuración.

### b.6. Sufijo de nombre y correo por igualdad

```sql
SELECT * FROM empleado
WHERE nombre LIKE '%Maria'
  AND e_mail = 'Marise@gmail.com';
```

El comodín inicial impide delimitar un rango de prefijo útil en un B+ normal por nombre. La propuesta compartida usa **e_mail** para localizar candidatos y después verifica el sufijo. La segunda columna no convierte mágicamente el sufijo en una búsqueda por prefijo.

Un índice simple por correo también puede ser suficiente si deja pocos candidatos. No hace falta introducir estructuras ajenas al programa para justificar esta solución.

### Resumen de a): conjunto de candidatos compartidos

| Consulta | Decisión justificada |
|---|---|
| b.1 | Evaluar entrega(fecha_entrega) para una interpretación directa por límite de fecha; conservar las diferencias de MONTHS_BETWEEN. |
| b.2 | Aprovechar el índice de PK existente; no duplicarlo. |
| b.3 | Evaluar tipo según distribución; Bitmap es candidato conceptual en un motor y carga adecuados. |
| b.4–b.6 | Un candidato compartido (e_mail,nombre), con clase de patrón si corresponde; alternativa más pequeña: solo e_mail. |

No crear a la vez todas las variantes. Elegir según índices previos, filas candidatas, planes y costo de mantenimiento. CREATE INDEX se comprobó solo en la instancia temporal para las propuestas PostgreSQL, sin alterar las bases de estudio.

### Respuesta a b): ¿para cuál se justifica Bitmap?

**b.3, tipo='N', es el candidato natural**, por baja cardinalidad relativa y consulta por igualdad. La elección se condiciona a tabla grande, carga mayormente de lectura, pocas modificaciones concurrentes y soporte del motor. Baja cardinalidad no garantiza por sí sola que sea mejor que leer la tabla.

Para b.1 y b.2 se estudian rangos con B+; nombre/correo tienen otro perfil y se benefician de acceso selectivo y del compuesto compartido. No confundir esta comparación de estructuras con los nodos Bitmap que PostgreSQL puede usar sobre índices ordinarios.

## Comprobaciones realizadas y alcance

- Se verificaron siete pares de resultados con EXCEPT ALL en ambos sentidos: 1.a, 1.b, 2, 3.a, 3.b y dos alternativas de 4.
- Se repitió 1.a con un caso de terror no vacío y se comprobó además la reescritura de 4 que conserva el discriminante, incluso con datos incoherentes del subtipo.
- Se contrastaron ambos órdenes de PK del ejercicio 3 en esquemas distintos, sin alterar una PK existente.
- Se comprobaron sintaxis de las seis consultas de 5 en sus variantes PostgreSQL y cuatro índices candidatos en la copia. No se ejecutó el SQL Oracle ni se creó un Bitmap persistente.
- Se comprobaron 32 bloques PostgreSQL de los dos apuntes finales; el bloque Oracle se revisó por su semántica, sin ejecutarlo.
- Las pruebas de equivalencia sobre datos complementan la justificación por restricciones; no la sustituyen. Los datos mínimos no permiten concluir qué índice será más rápido con tablas grandes.

## Fuentes

Material de cátedra dentro de Bases de Datos, fuera del baúl:

- Practicos/Practico 7 - Optimización de Consultas.docx, con ambos diagramas revisados.
- Contenido extra/Filminas y Practicos de cursada/2023/BDI_07 (Optimizacion).pdf.
- Scripts de estructura y datos de práctica en FinalBases/catedra: peliculas.sql, peliculas_datos_tp2.sql, voluntarios.sql y voluntarios_datos_tp2.sql. Se usaron copias en una instancia temporal.

Detalles del motor: [EXPLAIN](https://www.postgresql.org/docs/15/using-explain.html), [índices compuestos](https://www.postgresql.org/docs/current/indexes-multicolumn.html), [tipos y LIKE](https://www.postgresql.org/docs/15/indexes-types.html), [Bitmap en ejecución](https://www.postgresql.org/docs/15/indexes-bitmap-scans.html), [Oracle MONTHS_BETWEEN](https://docs.oracle.com/database/121/SQLRF/functions114.htm) y [Oracle Bitmap](https://docs.oracle.com/en/database/oracle/oracle-database/21/tgdba/designing-and-developing-for-performance.html).

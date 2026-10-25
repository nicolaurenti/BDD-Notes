---
tags:
  - tp2
---

# Joins y conteos de filas relacionadas

## Diferencias entre joins

| Join | Filas que conserva |
|---|---|
| `INNER JOIN` | Solo las que encuentran coincidencia. |
| `LEFT JOIN` | Todas las de la izquierda y sus coincidencias. |
| `RIGHT JOIN` | Todas las de la derecha y sus coincidencias. |
| `FULL JOIN` | Todas las de ambos lados. |
| `CROSS JOIN` | Todas las combinaciones posibles. |
| `NATURAL JOIN` | Combina usando todas las columnas con el mismo nombre. |

Cuando un outer join no encuentra coincidencia, completa las columnas del lado faltante con `NULL`.

### INNER y LEFT con instituciones

INNER conserva coincidencias; LEFT conserva además las filas izquierdas sin pareja, completando el otro lado con NULL.

### Ejemplos básicos de INNER JOIN, LEFT JOIN y USING

Consultas básicas de instituciones y voluntarios para contrastar coincidencias y filas preservadas.

#### INNER JOIN

```sql
SELECT i.nombre_institucion, v.nombre, v.apellido
FROM institucion i
JOIN voluntario v
    ON v.id_institucion = i.id_institucion;
```

Solo aparecen instituciones con voluntarios relacionados.

#### LEFT JOIN

```sql
SELECT i.nombre_institucion, v.nombre, v.apellido
FROM institucion i
LEFT JOIN voluntario v
    ON v.id_institucion = i.id_institucion;
```

También aparecen las instituciones sin voluntarios.

#### JOIN USING

Cuando las columnas de unión tienen el mismo nombre:

```sql
FROM voluntario v
JOIN institucion i USING (id_institucion)
```

> [!warning]
> `NATURAL JOIN` utiliza todas las columnas con nombres coincidentes. Si cambia la estructura de las tablas, puede cambiar también el significado del join.

### Filtros en un LEFT JOIN

ON decide qué coincidencias se forman. WHERE filtra después de la unión y puede quitar las filas extendidas con NULL que se querían conservar.

### LEFT JOIN con filtros en ON y WHERE

Compará las dos consultas y seguí sus resultados con instituciones 20 y 50.

#### Diferencia entre ON y WHERE

Para conservar todas las instituciones y mostrar solamente sus voluntarios con más de 1000 horas:

```sql
SELECT i.id_institucion, v.nro_voluntario
FROM institucion i
LEFT JOIN voluntario v
    ON v.id_institucion = i.id_institucion
   AND v.horas_aportadas > 1000;
```

Si esa condición se coloca en `WHERE`, desaparecen las filas sin una coincidencia que la cumpla:

```sql
SELECT i.id_institucion, v.nro_voluntario
FROM institucion i
LEFT JOIN voluntario v
    ON v.id_institucion = i.id_institucion
WHERE v.horas_aportadas > 1000;
```

#### Una tabla pequeña para seguir el LEFT JOIN

Sean instituciones 20 y 50, y dos voluntarios de 20: `(1, 500 horas)` y `(2, 2000 horas)`. El LEFT JOIN sin otros filtros produce:

| Institución | Voluntario | Horas |
|---|---|---:|
| 20 | 1 | 500 |
| 20 | 2 | 2000 |
| 50 | NULL | NULL |

Si `horas > 1000` está en ON, las coincidencias se buscan con ese requisito: salen `(20, 2, 2000)` y `(50, NULL, NULL)`. Si está en WHERE, se filtra después de unir: de la tabla anterior solo queda `(20, 2, 2000)`, porque NULL > 1000 no es TRUE.

Al agrupar el join original por la clave de la institución, COUNT(*) produce 2 para 20 y 1 para 50. COUNT de la PK del voluntario produce 2 y 0. La diferencia proviene de la fila que LEFT JOIN conserva para representar la institución sin coincidencias.

En las uniones por departamento de Películas, usar `(id_distribuidor, id_departamento)`. Igualar solo id_departamento puede unir departamentos diferentes que comparten número y multiplicar filas sin que corresponda al modelo.

### Relacionar una tabla consigo misma

Un autojoin usa alias distintos para representar roles diferentes de una misma tabla, como voluntario y coordinador.

### Autojoins, coordinadores y parejas de filas

Incluye el caso voluntario–coordinador y el efecto de nulos y valores repetidos al emparejar equipos.

#### Autojoin

Permite relacionar una tabla consigo misma utilizando alias diferentes:

```sql
SELECT
    v.nombre,
    v.apellido,
    c.nombre AS nombre_coordinador,
    c.apellido AS apellido_coordinador
FROM voluntario v
LEFT JOIN voluntario c
    ON c.nro_voluntario = v.id_coordinador;
```

#### Autojoin: todas las coincidencias y efecto de NULL

```sql
SELECT e1.id AS id_1, e2.id AS id_2
FROM equipo e1
JOIN equipo e2 ON e1.puntos = e2.puntos;
```

- Con cuatro equipos cuyos puntos son `NULL`, devuelve **0 filas**: `NULL = NULL` es `UNKNOWN` y no forma una coincidencia.
- Con cuatro equipos cuyos puntos son todos `5`, devuelve **16 filas**: cada fila coincide con las cuatro, incluida ella misma. Aparecen tanto `(1, 2)` como `(2, 1)`.

El join no elige una única coincidencia por fila: devuelve **todas las parejas que satisfacen el ON**. Los alias representan dos usos de la misma tabla.

Los nulos y los conteos del join se explican en esta nota y en el bloque de NULL del TP2.

### USING y nombres comunes

USING permite indicar la columna común cuando tiene el mismo nombre. NATURAL JOIN usa todas las columnas con nombres coincidentes, por lo que debe revisarse cuidadosamente.

### Cantidad de parejas de un autojoin

El join produce todas las parejas que satisfacen ON, incluso la pareja de una fila consigo misma si no se excluye.

## Contar filas relacionadas

Para contar entidades después de unir, identificar qué representa cada fila. En LEFT JOIN se cuenta la PK del lado opcional para obtener cero cuando no hay coincidencias.

### Conteos de entidades relacionadas y ausentes

Consultas para incluir entidades sin relacionados y evitar multiplicaciones.

#### Conteos de joins

#### Conservar entidades sin relacionados

Cantidad de voluntarios de cada institución, incluyendo las que tienen cero:

```sql
SELECT
    i.id_institucion,
    COUNT(v.nro_voluntario) AS cantidad
FROM institucion i
LEFT JOIN voluntario v
    ON v.id_institucion = i.id_institucion
GROUP BY i.id_institucion;
```

Se cuenta la PK del voluntario:

- Sin voluntarios, `COUNT(v.nro_voluntario)` devuelve `0`.
- Con `COUNT(*)`, se contaría la fila extendida con nulos y devolvería `1`.

#### Evitar multiplicaciones

Unir un voluntario con varias filas históricas genera varias filas para ese voluntario.

Si se quieren contar personas diferentes:

```sql
COUNT(DISTINCT v.nro_voluntario)
```

> [!important]
> Antes de agregar, identificar qué representa cada fila del resultado del join: una persona, una tarea, una entrega o una combinación.

## Resultado trabajado de LEFT JOIN

El desarrollo muestra las filas concretas antes y después de filtrar, y por qué COUNT(*) difiere de contar la PK opcional.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).

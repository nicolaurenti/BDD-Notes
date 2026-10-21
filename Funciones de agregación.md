---
tags:
  - tp2
---

# Agregaciones, agrupamiento y porcentajes

## Funciones de agregación

| Función | Resultado |
|---|---|
| `COUNT(*)` | Cantidad de filas. |
| `COUNT(columna)` | Cantidad de valores no nulos de la columna. |
| `COUNT(DISTINCT columna)` | Cantidad de valores diferentes y no nulos. |
| `SUM(columna)` | Suma de valores no nulos. |
| `AVG(columna)` | Promedio de valores no nulos. |
| `MIN(columna)` | Menor valor no nulo. |
| `MAX(columna)` | Mayor valor no nulo. |
| `STDDEV(columna)` | Desviación estándar muestral de los valores no nulos. |

#### Diferencias cuando todos los valores son NULL

Si existen cuatro filas y todas tienen `puntos = NULL`:

| Expresión | Resultado |
|---|---|
| `COUNT(*)` | `4` |
| `COUNT(puntos)` | `0` |
| `AVG(puntos)` | `NULL` |
| `SUM(puntos)` | `NULL` |

Sin filas de entrada y sin `GROUP BY`, `COUNT` devuelve `0`; las demás agregaciones de esta tabla devuelven `NULL`.

#### COALESCE: reemplazar el resultado o los datos de entrada

Para mostrar cero cuando el promedio es nulo (ejercicio 2.1.b):

```sql
SELECT COALESCE(AVG(puntos), 0) AS promedio,
       COUNT(puntos) AS cantidad_puntos,
       COUNT(*) AS cantidad_filas
FROM equipo;
```

Con cuatro filas cuyos puntos son nulos devuelve `0 | 0 | 4`. Los conteos ya devuelven cero cuando no hay valores para contar; no necesitan `COALESCE`.

Estas expresiones tienen significados diferentes:

```sql
COALESCE(AVG(puntos), 0) -- Sustituye el resultado si el promedio es NULL.
AVG(COALESCE(puntos, 0)) -- Convierte cada punto NULL en cero antes de promediar.
```

Con puntos `10, NULL`, la primera devuelve **10** y la segunda **5**. Sobre una entrada sin filas, la segunda sigue devolviendo `NULL`, porque no hay filas sobre las que aplicar `COALESCE`.

Si se necesita un blanco en la salida, hay que convertir el promedio a texto:

```sql
SELECT COALESCE(CAST(AVG(puntos) AS text), '') AS promedio
FROM equipo;
```

El bloque de NULL del TP2 desarrolla la lógica de tres valores.

## GROUP BY y HAVING

`GROUP BY` reúne filas que tienen los mismos valores en las columnas indicadas.

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
GROUP BY id_institucion;
```

Como regla general, cada expresión seleccionada debe estar agrupada o calculada mediante una agregación.

#### WHERE y HAVING

- `WHERE`: filtra filas antes de agrupar.
- `HAVING`: filtra grupos después de calcular las agregaciones.

Instituciones con más de diez voluntarios que aportaron más de 1000 horas:

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
WHERE horas_aportadas > 1000
GROUP BY id_institucion
HAVING COUNT(*) > 10;
```

#### Agrupar por varias columnas

Cantidad de voluntarios por institución y coordinador:

```sql
SELECT
    id_institucion,
    id_coordinador,
    COUNT(*) AS cantidad
FROM voluntario
WHERE id_coordinador IS NOT NULL
GROUP BY id_institucion, id_coordinador
HAVING COUNT(*) < 3;
```

#### Agrupar por identificadores

Agrupar solamente por nombre puede mezclar entidades diferentes que se llaman igual.

Conviene agrupar por la clave y, cuando corresponda, por las columnas descriptivas seleccionadas.

## Promedio de filas que cumplen una condición

#### Filtrar antes de calcular el promedio

Promedio de horas de los voluntarios que aportaron más de 1000 horas:

```sql
SELECT AVG(horas_aportadas)
FROM voluntario
WHERE horas_aportadas > 1000;
```

#### Conservar el total y calcular un promedio de un subconjunto

El recurso básico del TP es separar las consultas: una cuenta todas las filas y otra filtra las que entran al promedio. Para una institución concreta:

```sql
SELECT AVG(horas_aportadas)
FROM voluntario
WHERE id_institucion = 20
  AND horas_aportadas > 1000;
```

No reemplazar por cero las horas que no cumplen: eso las haría participar del promedio. Si en esa institución hay horas 500, 2000 y 3000, el promedio solicitado es `(2000 + 3000) / 2 = 2500`, no `5000 / 3`.

#### Promedio y porcentaje responden preguntas distintas

El promedio anterior mide horas. Para medir qué proporción de voluntarios supera 1000 horas se cuentan personas:

```sql
SELECT 100.0 * (
    SELECT COUNT(*)
    FROM voluntario
    WHERE horas_aportadas > 1000
) / NULLIF(COUNT(*), 0) AS porcentaje
FROM voluntario;
```

El numerador cuenta los casos que cumplen y el denominador cuenta todos. `100.0` permite un resultado decimal; NULLIF devuelve NULL si el total es cero y evita la división inválida. Con 2 casos sobre 3, se obtiene aproximadamente 66.67 %. Las horas NULL no cumplen el filtro del numerador, pero sus filas sí integran COUNT(*) del denominador; cambiar eso requiere una decisión explícita.

## Comparación con totales y porcentajes

Instituciones con menos del 10 % de los voluntarios registrados:

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
WHERE id_institucion IS NOT NULL
GROUP BY id_institucion
HAVING COUNT(*) < (
    SELECT COUNT(*) * 0.10
    FROM voluntario
);
```

La subconsulta calcula el total general; el agrupamiento calcula la cantidad por institución.

> [!important]
> Definir correctamente el denominador: total de personas, total de tareas distintas o total de registros no significan lo mismo.

Este ejemplo considera instituciones con al menos un voluntario. Para incluir las que tienen cero, hay que partir de `institucion` y utilizar un `LEFT JOIN`.

## Contar lo que pide el enunciado

Si un voluntario tiene tres registros históricos con tareas A, A y B, `COUNT(*)` da 3 y `COUNT(DISTINCT id_tarea)` da 2. La primera cantidad representa registros o períodos; la segunda, tareas diferentes. Ninguna demuestra por sí sola cuántas veces cambió de tarea: para hablar de cambios debe estar definido cómo cada registro representa un período o una transición.

En una institución con voluntarios `(coordinador 10, persona 1)`, `(10, 2)` y `(20, 3)`, agrupar solo por institución produce cantidad 3. Agrupar por institución y coordinador produce los grupos de tamaños 2 y 1. Para «coordinadores con menos de 3 voluntarios dentro de cada institución», la clave del grupo debe contener ambas columnas.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).

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

Ver Valores NULL y lógica de tres valores.

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

#### Promedio condicional conservando otros datos

```sql
SELECT
    id_institucion,
    COUNT(*) AS total_voluntarios,
    AVG(horas_aportadas)
        FILTER (WHERE horas_aportadas > 1000)
        AS promedio_mayores_a_1000
FROM voluntario
GROUP BY id_institucion;
```

`FILTER` afecta solamente a la agregación donde aparece.

#### Alternativa con CASE

```sql
SELECT AVG(
    CASE
        WHEN horas_aportadas > 1000
        THEN horas_aportadas
    END
)
FROM voluntario;
```

Cuando no se cumple la condición, el `CASE` devuelve `NULL`, que `AVG` ignora.

> [!warning]
> Agregar `ELSE 0` cambia el cálculo: las filas que no cumplen la condición pasan a participar del promedio como ceros.

#### Promedio y porcentaje no son lo mismo

Para calcular el porcentaje de voluntarios que cumplen una condición:

```sql
SELECT
    100.0 * COUNT(*) FILTER (WHERE horas_aportadas > 1000)
    / NULLIF(COUNT(*), 0) AS porcentaje
FROM voluntario;
```

- `100.0` evita la división entera.
- `NULLIF` evita dividir por cero.

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

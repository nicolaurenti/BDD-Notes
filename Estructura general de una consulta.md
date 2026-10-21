---
tags:
  - tp2
---

# Estructura, ejecución y presentación de resultados

## Estructura general de una consulta

```POSTGRESQL
SELECT columnas
FROM tabla
WHERE condicion
GROUP BY columnas
HAVING condicion_sobre_grupos
ORDER BY columnas
LIMIT cantidad
OFFSET desplazamiento;
```

| Cláusula      | Función                                          |
| ------------- | ------------------------------------------------ |
| `SELECT`      | Define las columnas o expresiones del resultado. |
| `FROM`        | Indica las tablas de origen.                     |
| `JOIN ... ON` | Combina filas de tablas según una condición.     |
| `WHERE`       | Filtra filas antes del agrupamiento.             |
| `GROUP BY`    | Forma grupos de filas.                           |
| `HAVING`      | Filtra grupos.                                   |
| `ORDER BY`    | Ordena el resultado.                             |
| `LIMIT`       | Limita la cantidad de filas devueltas.           |
| `OFFSET`      | Omite una cantidad de filas.                     |

## Orden lógico de ejecución

El orden lógico para interpretar una consulta es:

1. `FROM` y `JOIN`: construir las filas de origen.
2. `WHERE`: filtrar filas.
3. `GROUP BY`: formar grupos.
4. Calcular las agregaciones.
5. `HAVING`: filtrar grupos.
6. `SELECT`: producir las columnas del resultado.
7. `DISTINCT`: eliminar filas repetidas.
8. `ORDER BY`: ordenar.
9. `OFFSET` y `LIMIT`: seleccionar la porción del resultado.

> [!important]
> Este es el orden lógico. El motor puede utilizar otro orden físico para ejecutar la consulta de manera más eficiente.

#### Consecuencias

- `WHERE` no puede filtrar por una agregación calculada en ese mismo nivel.
- Los alias definidos en `SELECT` pueden usarse en `ORDER BY`, pero no en `WHERE` ni en `HAVING` en PostgreSQL.

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
GROUP BY id_institucion
HAVING COUNT(*) > 10
ORDER BY cantidad DESC;
```

## DISTINCT

Elimina filas repetidas del resultado.

```sql
SELECT DISTINCT id_tarea
FROM voluntario;
```

Si se seleccionan varias columnas, se consideran las **combinaciones completas**:

```sql
SELECT DISTINCT id_institucion, id_tarea
FROM voluntario;
```

Una misma institución puede aparecer varias veces si está asociada con tareas diferentes.

#### COUNT con DISTINCT

```sql
SELECT COUNT(DISTINCT id_tarea)
FROM voluntario;
```

Cuenta tareas diferentes, excluyendo los valores nulos.

> [!warning]
> `DISTINCT` no corrige un join mal planteado. Primero hay que comprobar por qué se repiten las filas.

#### DISTINCT y valores nulos

Con cuatro filas cuyos puntos son nulos:

```sql
SELECT DISTINCT puntos FROM equipo;
```

Devuelve **una fila con `NULL`**. En cambio, `COUNT(DISTINCT puntos)` devuelve **0**, porque el conteo excluye los nulos.

La eliminación de duplicados no usa el mismo criterio que una comparación ordinaria: `NULL = NULL` sigue dando `UNKNOWN`.

El bloque de NULL del TP2 desarrolla la lógica de tres valores.

## Ordenamiento, primeros N y paginación

#### Ordenamiento

```sql
SELECT nombre, apellido, horas_aportadas
FROM voluntario
ORDER BY horas_aportadas DESC, nro_voluntario ASC;
```

- `ASC`: ascendente.
- `DESC`: descendente.
- Los criterios posteriores resuelven empates de los anteriores.
- `NULLS FIRST` y `NULLS LAST` permiten ubicar explícitamente los nulos.

#### Primeros N

```sql
SELECT *
FROM voluntario
ORDER BY horas_aportadas DESC NULLS LAST,
         nro_voluntario
LIMIT 5;
```

Devuelve cinco filas como máximo. Si hay empate en el límite, no necesariamente incluye a todos los empatados.

#### Paginación

```text
OFFSET = (número de página - 1) × tamaño de página
```

Tercera página con cinco registros por página:

```sql
SELECT *
FROM tarea
ORDER BY id_tarea
LIMIT 5
OFFSET 10;
```

> [!warning]
> Para paginar de forma estable se necesita un `ORDER BY` que determine un orden completo, utilizando una clave para resolver empates.

## Seguir una consulta con datos pequeños

Supongamos estos voluntarios, mostrados como `(nro, institución, horas)`: `(1, 20, 500)`, `(2, 20, 2000)`, `(3, 20, 3000)`, `(4, 50, 4000)`.

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
WHERE horas_aportadas > 1000
GROUP BY id_institucion
HAVING COUNT(*) >= 2
ORDER BY id_institucion;
```

WHERE elimina al voluntario 1. Quedan dos filas de la institución 20 y una de la 50. GROUP BY forma esos dos grupos; COUNT produce 2 y 1; HAVING conserva solo el grupo de 20. El resultado es una fila `(20, 2)`.

Si se quitara WHERE, la cantidad para 20 sería 3. Si se cambiara HAVING por `WHERE COUNT(*) >= 2`, la consulta sería incorrecta: WHERE decide sobre filas antes de formar los grupos. Escribir primero qué representa una fila de entrada y qué debe representar una fila de salida ayuda a elegir las cláusulas.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).

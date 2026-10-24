---
tags:
  - tp2
---

# TP2 - Promedios, porcentajes y comparación con totales

Incluye promedios de un subconjunto, porcentajes y cantidades por institución frente al total general.

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

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).

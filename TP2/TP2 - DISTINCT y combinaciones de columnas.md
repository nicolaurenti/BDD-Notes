---
tags:
  - tp2
---

# TP2 - DISTINCT y combinaciones de columnas

Los ejemplos distinguen filas diferentes, conteo de valores distintos y tratamiento de NULL.

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

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).

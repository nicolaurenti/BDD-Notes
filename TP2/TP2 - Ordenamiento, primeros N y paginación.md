---
tags:
  - tp2
---

# TP2 - Ordenamiento, primeros N y paginación

Incluye criterios de orden, empates y el cálculo de la tercera página de cinco filas.

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

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).

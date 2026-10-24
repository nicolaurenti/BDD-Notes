---
tags:
  - tp2
---

# TP2 - Autojoins, coordinadores y parejas de filas

Incluye el caso voluntario–coordinador y el efecto de nulos y valores repetidos al emparejar equipos.

## Autojoin

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

## Autojoin: todas las coincidencias y efecto de NULL

```sql
SELECT e1.id AS id_1, e2.id AS id_2
FROM equipo e1
JOIN equipo e2 ON e1.puntos = e2.puntos;
```

- Con cuatro equipos cuyos puntos son `NULL`, devuelve **0 filas**: `NULL = NULL` es `UNKNOWN` y no forma una coincidencia.
- Con cuatro equipos cuyos puntos son todos `5`, devuelve **16 filas**: cada fila coincide con las cuatro, incluida ella misma. Aparecen tanto `(1, 2)` como `(2, 1)`.

El join no elige una única coincidencia por fila: devuelve **todas las parejas que satisfacen el ON**. Los alias representan dos usos de la misma tabla.

Los nulos y los conteos del join se explican en esta nota y en el bloque de NULL del TP2.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
